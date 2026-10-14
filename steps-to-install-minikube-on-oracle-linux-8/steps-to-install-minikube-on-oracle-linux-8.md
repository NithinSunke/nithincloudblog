---
title: "Steps to Install Minikube on Oracle Linux 8"
author: "Sunke Nithin"
date: 2025-09-06
categories: ["Kubernetes"]
tags: ["Devops", "Linux", "podman"]
---

# Steps to Install Minikube on Oracle Linux 8

Minikube is a popular tool that allows developers and system administrators to run a single-node Kubernetes cluster locally. This is especially useful for development, testing, and learning Kubernetes without setting up a full multi-node cluster. This guide walks through installing Minikube on Oracle Linux 8 (OEL8) using the Podman container runtime.

## Prerequisites

Before starting the installation, ensure that your Oracle Linux 8 system meets these requirements:

- A non-root user with sudo privileges.
- Podman and cURL packages installed.
- Up-to-date system packages.

## Step 1: Prepare the System

Start by updating the system and installing necessary packages including container tools and Podman:

``` wp-block-code
sudo dnf -y module install container-tools:ol8
sudo dnf -y install conntrack podman curl
sudo dnf -y update
```

Set Python 3 to use Python 3.9 (needed for tools compatibility):

``` wp-block-code
sudo alternatives --set python3 /usr/bin/python3.9
```

## Step 2: Download and Install Minikube

Download the latest Minikube binary directly from the official storage:

``` wp-block-code
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

Install the binary:

``` wp-block-code
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Clean up the downloaded binary:

``` wp-block-code
rm minikube-linux-amd64
```

## Step 3: Configure Minikube with Podman Driver

Configure Minikube to use Podman as the container runtime driver:

``` wp-block-code
minikube config set driver podman
```

## Step 4: Start Minikube

Start your Minikube cluster with desired resources. The example below allocates maximum CPUs with 7.9GB memory and a 40GB disk size. It also specifies using the CRI-O container runtime:

``` wp-block-code
minikube start --cpus max --memory 7900mb --disk-size='40g' --container-runtime=cri-o
```

## Step 5: Enable Useful Add-ons (Optional)

Enable the Kubernetes Ingress add-on to allow external access to your services:

``` wp-block-code
minikube addons enable ingress
```

## Step 6: Verify Installation

Check Minikube status to ensure the cluster is running:

``` wp-block-code
minikube status
```

Verify Minikube can communicate with Kubernetes:

``` wp-block-code
minikube kubectl -- get po -A
kubectl version
```

if kubectl is not installed install

``` wp-block-code
VERSION=$(curl -sL https://dl.k8s.io/release/stable.txt)
curl -LO https://dl.k8s.io/release/${VERSION}/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
kubectl version
```

to stop minikube

``` wp-block-code
minikube stop
minikube status
```

to Delete the single node cluster

``` wp-block-code
minikube delete
```

to get kubernetes nodes list

``` wp-block-code
 kubectl get nodes
```

## Conclusion

With these steps, you have successfully installed Minikube on Oracle Linux 8 using the Podman driver. This setup provides a powerful local Kubernetes cluster to develop and test your containerized applications with an environment closely mimicking production.
