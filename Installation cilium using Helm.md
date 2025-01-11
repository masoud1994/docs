# Installation cilium using Helm
Introduction to Cilium & Hubble

## questions?
whats diffrent between k8s ipam and clilum ipam and how to change cidr in cilium?
* https://docs.cilium.io/en/stable/network/concepts/ipam/

## What is Cilium?
* https://docs.cilium.io/en/stable/overview/intro/

Cilium is open source software for transparently securing the network connectivity between application services deployed using Linux container management platforms like Docker and Kubernetes.

At the foundation of Cilium is a new Linux kernel technology called eBPF, which enables the dynamic insertion of powerful security visibility and control logic within Linux itself. Because eBPF runs inside the Linux kernel, Cilium security policies can be applied and updated without any changes to the application code or container configuration.

This guide will show you how to install Cilium using Helm. This involves a couple of additional steps compared to the [Cilium Quick Installation](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#k8s-quick-install) and requires you to manually select the best datapath and IPAM mode for your particular environment.

## Why Cilium & Hubble?

This shift toward highly dynamic microservices presents both a challenge and an opportunity in terms of securing connectivity between microservices. Traditional Linux network security approaches (e.g., iptables) filter on IP address and TCP/UDP ports, but IP addresses frequently churn in dynamic microservices environments. The highly volatile life cycle of containers causes these approaches to struggle to scale side by side with the application as load balancing tables and access control lists carrying hundreds of thousands of rules that need to be updated with a continuously growing frequency. Protocol ports (e.g. TCP port 80 for HTTP traffic) can no longer be used to differentiate between application traffic for security purposes as the port is utilized for a wide range of messages across services.
    
## Install Cilium
there are diffrent way to install cilium and it depends on kuberenetes.
In this case we explain install cilium on rke , for mor information see below urls.
* https://docs.cilium.io/en/stable/installation/k8s-install-helm/
* https://docs.cilium.io/en/stable/installation/k8s-install-rke/#deploy-cilium
Setup Helm repository:

```
helm repo add cilium https://helm.cilium.io
helm repo update
helm install cilium cilium/cilium --version 1.16.5 \
   --namespace $CILIUM_NAMESPACE
```

## install the Cilium Helm chart offline
To install the Cilium Helm chart offline, you'll need to download the necessary Helm chart and all of its dependencies, and then install it from your local machine. This process involves downloading the Cilium Helm chart, packaging it, and installing it without needing an internet connection during the actual installation. Here's how you can do this:
## Step 1: Download the Cilium Helm Chart and Its Dependencies Ensure 
### Helm is installed and configured:
* https://helm.sh/docs/intro/quickstart/

helm completion bash

Generate the autocompletion script for Helm for the bash shell.

To load completions in your current shell session:
```
source <(helm completion bash)

```
To load completions for every new session, execute once:

Linux:
```
helm completion bash > /etc/bash_completion.d/helm
```
```
If you haven't already installed Helm, follow the instructions mentioned earlier to install it.
### Add the Cilium Helm repository:
```
helm repo add cilium https://helm.cilium.io/
```
### Update your Helm repositories:
```
helm repo update
```
### Download the Cilium Helm chart:
Now, you will need to download the Cilium chart and its dependencies. You can use the helm pull command to download the chart and optionally save it as a .tgz file.
```
helm pull cilium/cilium --version <chart-version> --untar
```
Replace <chart-version> with the version of the Cilium chart you want to download, or leave it out to get the latest version.

This will create a folder called cilium containing the chart files.

### Download dependencies:
If there are dependencies for the Cilium chart, you can download them using:

```
helm dependency update ./cilium
```
After downloading the chart and its dependencies, you can package it into a .tgz file for offline installation.
```
helm package ./cilium
```

```
helm upgrade --install  cilium . -n kube-system \
  --set ipam.mode=cilium \
  --set ipam.operator.podCIDR=10.42.0.0/16
  ```