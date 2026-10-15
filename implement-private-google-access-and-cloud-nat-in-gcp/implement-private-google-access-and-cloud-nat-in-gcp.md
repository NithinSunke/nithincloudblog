---
title: "Implement Private Google Access and Cloud NAT in GCP"
author: "Sunke Nithin"
date: 2025-08-11
categories: ["GCP"]
tags: ["AWS", "AZURE", "CLOUD", "security", "technology"]
source: https://nithinsunke.wordpress.com/2025/08/11/implement-private-google-access-and-cloud-nat-in-gcp/
---

# Implement Private Google Access and Cloud NAT in GCP

**Goal:** allow VM instances *without external IPs* to (a) reach Google APIs privately (Private Google Access) and (b) reach the public internet via Cloud NAT for other outbound traffic — securely and scalably.

**When to use:** private workloads (no external IPs) that still need to contact Google APIs (Cloud Storage, BigQuery, etc.) and/or public internet endpoints (package repos, external APIs) without exposing instance external IPs. See official docs for exact supported Google services. [Google Cloud](https://cloud.google.com/vpc/docs/private-google-access)

## High Level Steps:

- VPC with one or more subnets.
- VMs in those subnets **do not** have external IPs.
- Enable **Private Google Access** on the subnet(s) so VMs can reach Google APIs via Google’s network addresses.
- Create a **Cloud Router**, then a **Cloud NAT** gateway (regional) attached to that router to provide outbound NAT for other internet traffic.
- Optionally configure NAT logging and NAT Rules for advanced behavior. [Google Cloud](https://cloud.google.com/nat/docs)[Google Codelabs](https://codelabs.developers.google.com/codelabs/cloudnet-natrules)

## Prerequisites

- GCP project with billing enabled.
- `gcloud` CLI configured (`gcloud init` or `gcloud auth login`) and a project selected.
- IAM: you need `compute.networkAdmin` (or similar) privileges to create routers, NATs, and edit subnets.

## Step 1 — Create a VPC + subnet (Console or gcloud)

**Console:** VPC Network → Create VPC network → Custom subnet → create subnet(s).  
When creating VMs later, **don’t assign external IPs**.

**gcloud (example):**

``` wp-block-code
gcloud compute networks create private-net --subnet-mode=custom

gcloud compute networks subnets create private-subnet \
  --network=private-net \
  --region=us-central1 \
  --range=10.10.0.0/24
```

## Step 2 — Enable Private Google Access on the subnet

This setting is **per-subnet**. It exposes Google API IPs to instances that lack external IPs.

**Console:** VPC Network → VPC networks → click subnet → Edit → set **Private Google Access: On** → Save. [Google Cloud](https://cloud.google.com/vpc/docs/configure-private-google-access)

``` wp-block-code
gcloud compute networks subnets update private-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access
```

If you need to enable across many subnets, you can script the update for each subnet. (Common tip: list subnets and loop.)

## Step 3 — Create a Cloud Router (required for Cloud NAT)

Cloud NAT attaches to a Cloud Router (regional).

``` wp-block-code
gcloud compute routers create nat-router \
  --network=private-net \
  --region=us-central1 \
  --asn=65001
```

## Step 4 — Create Cloud NAT (console, gcloud, or Terraform)

You can let GCP auto-allocate external IPs or supply static external IP addresses. Common flags are `--nat-all-subnet-ip-ranges` (map all ranges) or `--source-subnet-ip-ranges` to be selective.

**gcloud (auto-allocated external IPs, NAT for all subnets in region):**

``` wp-block-code
gcloud compute routers nats create nat-gateway \
  --router=nat-router \
  --router-region=us-central1 \
  --auto-allocate-nat-external-ips \
  --nat-all-subnet-ip-ranges \
  --enable-logging
```

### Flags explained:

- `--auto-allocate-nat-external-ips`: GCP creates ephemeral external IPs for NAT.
- `--nat-all-subnet-ip-ranges`: apply NAT to all subnets in the router’s region.
- `--enable-logging`: enable Cloud NAT logging for troubleshooting. [Google Cloud](https://cloud.google.com/nat/docs/set-up-manage-network-address-translation)

**Console:** VPC Network → NAT (or Network services → Cloud NAT) → Get started → Create Cloud NAT (pick Router, region, NAT settings, logging).

**Notes:**

- You can instead create static external IP addresses and reference them with `--nat-external-ip-pool` or similar options for predictable egress IPs.
- For private NAT (no public egress), GCP also supports Private NAT (NAT to internal IP ranges); see docs if you need that behavior. [Google Cloud](https://cloud.google.com/nat/docs/set-up-private-nat)

## Step 5 — Verification

1.  Create a VM **without** external IP in `private-subnet`.
2.  SSH into a bastion (a separate VM with external IP) then SSH into private VM, or use IAP.
3.  From private VM:
    - Test Private Google Access: `curl -I https://storage.googleapis.com` or `curl -I https://www.googleapis.com` — these should succeed and route internally to Google API endpoints if PGA is enabled. [Google Cloud](https://cloud.google.com/vpc/docs/configure-private-google-access)
    - Test public internet egress: `curl -I https://example.com`. If Cloud NAT is configured correctly, this will succeed even though the VM has no external IP.
4.  Check Cloud NAT logs in Cloud Logging (filter for `resource.type="nat_gateway"`) if you enabled logging. [](https://cloud.google.com/nat/docs)

## Summary

- Create VPC & subnet (no external IPs for private VMs)
- Enable **Private Google Access** on subnet(s). (`gcloud compute networks subnets update ... --enable-private-ip-google-access`) [Google Cloud](https://cloud.google.com/vpc/docs/configure-private-google-access)
- Create **Cloud Router**. (`gcloud compute routers create ...`) [Google Cloud](https://cloud.google.com/nat/docs/set-up-manage-network-address-translation)
- Create **Cloud NAT** attached to router (auto or static IPs). (`gcloud compute routers nats create ...`) [Google Cloud](https://cloud.google.com/nat/docs/set-up-manage-network-address-translation)
- Verify from private VM: access Google APIs and public internet; check NAT logs. [Google Cloud](https://cloud.google.com/nat/docs)
