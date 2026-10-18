---
title: "Google Cloud VPC: A Complete Guide for Beginners"
author: "Sunke Nithin"
date: 2025-08-09
categories: ["GCP"]
tags: ["AWS", "AZURE", "CLOUD", "security", "technology"]
source: https://nithinsunke.wordpress.com/2025/08/09/google-cloud-vpc-a-complete-guide-for-beginners/
---

# Google Cloud VPC: A Complete Guide for Beginners

Virtual Private Cloud (VPC) is one of the foundational components of Google Cloud. It allows you to define your own **private network** within GCP, control how your resources communicate, and securely connect your infrastructure.

In this post, we’ll cover everything you need to know about **VPC in GCP**—concepts, components, best practices, and hands-on examples.

## What is a VPC?

A **Virtual Private Cloud (VPC)** in GCP is a **global, private, and isolated virtual network** that lets you run your cloud resources like Compute Engine VMs, Kubernetes clusters, and more.

Think of it as your **own datacenter in the cloud**, fully configurable with IP ranges, firewalls, routes, and peering.

## Core Components of a VPC

1.**Subnets**

- VPCs are **global**, but **subnets are regional**.
- You define **IP ranges** for each subnet.
- Subnets host your resources (like VMs, GKE clusters, etc.).

2\. **Routes**

- Define how traffic moves within and outside the VPC.
- Every VPC has **default routes**, like `0.0.0.0/0` pointing to the internet via an internet gateway.

3\. **Firewall Rules**

- Stateful rules to **allow or deny** traffic.
- Applied at the **network level**, not instance level.
- Supports `allow` (default), but **no `deny` rules** for egress.

4\. **Peering**

- Connect one VPC to another privately, across projects or organizations.

5\. **Shared VPC**

- Allows multiple projects to share a central VPC.
- Best for large organizations with separate billing/project needs.

6\. **Cloud NAT, VPN, Interconnect**

- **NAT**: Allow outbound internet access without exposing internal IPs.
- **VPN**: Securely connect on-premise to GCP.
- **Interconnect**: Dedicated physical connection to GCP.  

##  How to Create a VPC in GCP (Step-by-Step)

**Option 1: Using Console**

1.  Go to [VPC Network page](https://console.cloud.google.com/networking/networks).
2.  Click **"Create VPC Network"**.
3.  Enter:
    - Name: `my-custom-vpc`
    - Subnet creation mode: `Custom`
4.  Add subnets:
    - Region: `us-central1`
    - Subnet name: `web-subnet`
    - IP range: `10.10.10.0/24`
5.  Click **Create**.

**Option 2: Using gcloud CLI**

``` wp-block-code
gcloud compute networks create my-custom-vpc \
  --subnet-mode=custom

gcloud compute networks subnets create web-subnet \
  --network=my-custom-vpc \
  --region=us-central1 \
  --range=10.10.10.0/24
```

## Firewall Rules Example

``` wp-block-code
gcloud compute firewall-rules create allow-http \
  --network=my-custom-vpc \
  --allow=tcp:80 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server
```

This allows HTTP (port 80) traffic from anywhere to instances tagged with `http-server`.

## VPC Peering Example

``` wp-block-code
gcloud compute networks peerings create peer-1-2 \
  --network=vpc-1 \
  --peer-network=vpc-2 \
  --auto-create-routes
```

Use this when you need private communication between two VPCs (cross-project or same project).

## Key Concepts to Remember

| Concept        | Description                                                |
|----------------|------------------------------------------------------------|
| Global VPC     | A VPC is global, but its subnets are regional              |
| CIDR           | IP ranges like `10.0.0.0/16` used to define subnets        |
| Custom vs Auto | Custom: You define subnets. Auto: GCP creates default ones |
| Firewall Rules | Control traffic to/from instances                          |
| Shared VPC     | One VPC shared by multiple GCP projects                    |
| Cloud NAT      | Allow egress without assigning external IPs                |

## Best Practices

1.  **Always use Custom mode VPCs** (avoid default/auto unless testing).
2.  **Use separate subnets for each environment** (dev, test, prod).
3.  **Use tags and service accounts for firewall targeting**.
4.  **Apply firewall rules tightly** — no `0.0.0.0/0` unless required.
5.  **Use IAM to control who can modify VPC settings**.
6.  **Enable Flow Logs** to monitor traffic and troubleshoot.
7.  **Keep audit logs** for all network-related changes.

## Real-world Use Case Example

**Company Setup**:

- `prod-vpc` with subnets in `us-east1`, `europe-west1`.
- Separate subnet for frontend, backend, and database.
- Use Cloud NAT for secure outbound traffic.
- Use VPC Peering to connect `prod-vpc` and `monitoring-vpc`.

**Benefits**:

- Least privilege access via firewall rules.
- Centralized security management.
- Cost-effective bandwidth through peering.

## Diagram: VPC Structure

``` wp-block-code
                +-------------------------+
                |     VPC: my-vpc         |
                +-----------+-------------+
                            |
        +------------------+------------------+
        |                                     |
+---------------+                     +---------------+
| Subnet: web   |                     | Subnet: db    |
| us-central1   |                     | us-central1   |
| 10.10.10.0/24 |                     | 10.10.20.0/24 |
+---------------+                     +---------------+
```

## Summary



Google Cloud VPC is the backbone of your cloud network. It gives you complete control over how your applications communicate—both internally and with the outside world.
