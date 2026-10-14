---
title: "Understanding Kubernetes Pods"
author: "Sunke Nithin"
date: 2025-09-06
categories: ["Kubernetes"]
tags: [Devops]
---

# Understanding Kubernetes Pods

Kubernetes Pods are the smallest deployable units in the Kubernetes ecosystem. A Pod represents one or more containers that share storage, network, and a specification defining how to run the containers. Pods are fundamental building blocks for running containerized applications in Kubernetes.

## What is a Kubernetes Pod?

A Pod can be thought of as a logical host for containers that are tightly coupled and need to share resources. Each Pod runs on a node in the cluster and provides an isolated environment with a shared IP address and storage volumes for all containers included.

There are two common Pod usage scenarios:

- Running a single container in a Pod (most typical case).
- Running multiple cooperating containers that work together closely (such as a web server container paired with a content updater container).

Each Pod has its own network namespace, so containers within the Pod communicate easily, but Pods are isolated from each other.

Creating a Kubernetes Pod: A Simple Demo

## Create Pod using kubectl command

You can quickly create a Pod with a single container using the `kubectl run` command. Here is an example to deploy an NGINX web server Pod:

``` wp-block-code
kubectl run ngnix --image=docker.io/library/nginx:latest
kubectl get pods
kubectl describe pod ngnix
kubectl get pods -o wide
```

## Use YAML for Pods

**YAML** (Yet Another Markup Language) is the configuration format for Kubernetes resources. Writing pod specifications in YAML makes deployments reproducible, scalable, and easily managed with version control systems.

A basic pod YAML file consists of:

- **spec**: Describes the pod’s container configuration, including image and port.[](https://www.cncf.io/blog/2022/03/03/how-to-write-yaml-file-for-kubernetes/)
- **apiVersion**: Specifies the Kubernetes API version to use (“v1” for Pods).
- **kind**: Identifies the resource type (“Pod”).
- **metadata**: Includes pod name and labels.

**Example YAML Manifest**

**ngnix_pod.yml**

``` wp-block-code
apiVersion: v1
kind: Pod
metadata:
  name: ngnix
  labels:
    app: ngnix-pod
    tier: frontend
spec:
  containers:
  - name: ngnix
    image: docker.io/library/nginx:latest
```

**Step-by-Step Deployment**

**Apply the Manifest**

``` wp-block-code
kubectl apply -f ngnix_pod.yml
```

Kubernetes reads your YAML and creates the pod as specified.

**Check the Pod Status:**

``` wp-block-code
kubectl get pods
kubectl get pods -o wide
```

## Summary

Kubernetes Pods are the fundamental deployable units that encapsulate one or more containers, shared resources, and network. Creating Pods can be done imperatively with the `kubectl run` command or declaratively through YAML manifests.

Pods enable running containerized applications with ease, providing networking, storage sharing, and lifecycle management. This demo showed how to deploy a simple NGINX Pod
