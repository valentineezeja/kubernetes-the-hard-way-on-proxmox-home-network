# Provisioning a CA and Generating TLS Certificates

In this lab you will provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) using CloudFlare's PKI toolkit, [cfssl](https://github.com/cloudflare/cfssl), then use it to bootstrap a Certificate Authority, and generate TLS certificates for the following components: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy.

## Certificate Authority

In this section you will provision a Certificate Authority that can be used to generate additional TLS certificates.

On the `gateway-01` VM, generate the CA configuration file, certificate, and private key:

```bash
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "FR",
      "L": "Rennes",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Bretagne"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

Results:

```bash
ca-key.pem
ca.pem
```

## Client and Server Certificates

In this section you will generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes `admin` user.

### The Admin Client Certificate

On the `gateway-01` VM, generate the `admin` client certificate and private key:

```bash
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "FR",
      "L": "Rennes",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Bretagne"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```

Results:

```bash
admin-key.pem
admin.pem
```

### The Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

On the `gateway-01` VM, generate a certificate and private key for each Kubernetes worker node (you need to replace YOUR_EXTERNAL_IP by your domain name. Example:  `my-domain.com`):

```bash
EXTERNAL_IP=YOUR_DOMAIN_NAME

for id_instance in 0 1 2; do
cat > worker-${id_instance}-csr.json <<EOF
{
  "CN": "system:node:worker-${id_instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "FR",
      "L": "Rennes",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Bretagne"
    }
  ]
}
EOF

INTERNAL_IP=172.16.0.2${id_instance}

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=worker-${id_instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  worker-${id_instance}-csr.json | cfssljson -bare worker-${id_instance}
done
```

Results:

```bash
worker-0-key.pem
worker-0.pem
worker-1-key.pem
worker-1.pem
worker-2-key.pem
worker-2.pem
```

### The Controller Manager Client Certificate

On the `gateway-01` VM, generate the `kube-controller-manager` client certificate and private key:

```bash
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "FR",
      "L": "Rennes",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Bretagne"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

Results:

```bash
kube-controller-manager-key.pem
kube-controller-manager.pem
```

### The Kube Proxy Client Certificate

On the `gateway-01` VM, generate the `kube-proxy` client certificate and private key:

```bash
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "FR",
      "L": "Rennes",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Bretagne"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
```

Results:

```bash
kube-proxy-key.pem
kube-proxy.pem
```

### The Scheduler Client Certificate

On the `gateway-01` VM, generate the `kube-scheduler` client certificate and private key:

```bash
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "FR",
      "L": "Rennes",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Bretagne"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```

Results:

```bash
kube-scheduler-key.pem
kube-scheduler.pem
```

### The Kubernetes API Server Certificate

The `kubernetes-the-hard-way` static IP address (in this case, your domain) will be included in the list of subject alternative names for the Kubernetes API Server certificate. This will ensure the certificate can be validated by remote clients.

On the `gateway-01` VM, generate the Kubernetes API Server certificate and private key (you need to replace YOUR_EXTERNAL_IP by your domain name):

```bash
KUBERNETES_PUBLIC_ADDRESS=YOUR_DOMAIN_NAME

KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "FR",
      "L": "Rennes",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Bretagne"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,172.16.0.10,172.16.0.11,172.16.0.12,${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,${KUBERNETES_HOSTNAMES} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
```

> The Kubernetes API server is automatically assigned the `kubernetes` internal dns name, which will be linked to the first IP address (`10.32.0.1`) from the address range (`10.32.0.0/24`) reserved for internal cluster services during the [control plane bootstrapping](08-bootstrapping-kubernetes-controllers.md#configure-the-kubernetes-api-server) lab.

Results:

```bash
kubernetes-key.pem
kubernetes.pem
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as described in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

On the `gateway-01` VM, generate the `service-account` certificate and private key:

```bash
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "FR",
      "L": "Rennes",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Bretagne"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
```

Results:

```bash
service-account-key.pem
service-account.pem
```

## Distribute the Client and Server Certificates

Copy the appropriate certificates and private keys to each worker instance:

```bash
for instance in worker-0 worker-1 worker-2; do
  scp ca.pem ${instance}-key.pem ${instance}.pem root@${instance}:~/
done
```

Copy the appropriate certificates and private keys to each controller instance:

```bash
for instance in controller-0 controller-1 controller-2; do
  scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem root@${instance}:~/
done
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)
