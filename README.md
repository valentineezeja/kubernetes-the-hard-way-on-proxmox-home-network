# Kubernetes The Hard Way - Proxmox (KVM)

This tutorial walks you through setting up Kubernetes the hard way. This guide is not for someone looking for a fully automated tool to bring up a Kubernetes cluster. Kubernetes The Hard Way is optimized for learning, which means taking the long route to ensure you understand each task required to bootstrap a Kubernetes cluster.

> The results of this tutorial should not be viewed as production ready, and may receive limited support from the community, but don't let that stop you from learning!

Shout out to Kelsey Hightower who created the original guide, and DushanthaS who created the Proxmox-specific guide that requires a dedicated/static public IP address. However, not everyone has access to a dedicated public IP address, especially in home environments where IP addresses are dynamically assigned by Internet Service Providers (ISPs).

To address this, I have modified the setup to work with a dynamic home router IP address using Cloudflare DNS and DDNS (Dynamic DNS) services. This approach allows for a more accessible and cost-effective Kubernetes setup without the need for a static IP.


### My Solution

Here’s a brief overview of the steps I used to get my Kubernetes setup working with a dynamic home router IP address:

1. **Cloudflare DNS**:  
   - Create an account on Cloudflare and add your domain.  
   - Configure DNS settings to use Cloudflare's nameservers.
>
2. **DDNS Configuration**:  
    - Set up a DDNS service (I used DDclient) to update Cloudflare DNS records with your current public IP address, and each time a new IP is assigned.
    - Ensure the DDNS service runs as a daemon on the admin server to handle IP changes.
> 
3. **Kubernetes Configuration**:  
    - Modify Kubernetes setup scripts to use the domain managed by Cloudflare.     

By following these steps, you can achieve a functional Kubernetes setup using a dynamic IP address, making it easier to follow this guide without a dedicated/static public IP address.


## Overview of the Network Architecture

![architecture network](docs/images/architecture-network.png)

## Copyright

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

## Target Audience

The target audience for this tutorial is someone planning to support a production Kubernetes cluster and wants to understand how everything fits together, in particular if you want to use a Proxmox hypervisor and do not have a dedicated/static public IP address.

## Cluster Details

Kubernetes The Hard Way guides you through bootstrapping a highly available Kubernetes cluster with end-to-end encryption between components and RBAC authentication.

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.29.1
* [containerd](https://github.com/containerd/containerd) v1.7.13
* [coredns](https://github.com/coredns/coredns) v1.11.1
* [cni-plugins](https://github.com/containernetworking/plugins) v1.4.0
* [etcd](https://github.com/etcd-io/etcd) v3.5.12

## Labs

This tutorial assumes you have access to a Proxmox hypervisor with at least 25GB free RAM and 140GB free HDD/SSD. While a Proxmox server is used for basic infrastructure requirements the lessons learned in this tutorial can be applied to other platforms (ESXi, KVM, VirtualBox, ...).

* [Prerequisites](docs/01-prerequisites.md)
* [Installing the Client Tools](docs/02-client-tools.md)
* [Provisioning Compute Resources](docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](docs/04-certificate-authority.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/05-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/06-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/07-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Provisioning Pod Network Routes](docs/11-pod-network-routes.md)
* [Deploying the DNS Cluster Add-on](docs/12-dns-addon.md)
* [Smoke Test](docs/13-smoke-test.md)
* [Cleaning Up](docs/14-cleanup.md)
