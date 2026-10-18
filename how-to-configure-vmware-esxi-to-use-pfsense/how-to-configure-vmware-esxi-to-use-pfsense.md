---
title: "How to Configure VMware ESXi to Use PFSense"
author: "Sunke Nithin"
date: 2025-04-16
categories: ["FIREWALL"]
tags: []
source: https://nithinsunke.wordpress.com/2025/04/16/how-to-configure-vmware-esxi-to-use-pfsense/
---

# How to Configure VMware ESXi to Use PFSense

f you’re running a **home lab** or managing a virtualized environment with **VMware ESXi**, setting up **PFSense as a virtual firewall** is a fantastic way to manage network traffic, VPN, and firewall rules within your virtual infrastructure.

Let’s walk through the steps to deploy and configure **PFSense on VMware ESXi**.

i have a esxi running on vmware workstation. i have attached the host-only network that is configured from the previous blog post.

![](images/image-374.png)

![](images/image-376.png)

we have 3 physical NIC's

![](images/image-378.png)

create a virtual switch and assign a vnic as uplink

![](images/image-379.png)

![](images/image-380.png)

![](images/image-381.png)

now create port group and attach virtual switch to it.

![](images/image-382.png)

![](images/image-383.png)

![](images/image-384.png)

for this blog post i have created a windows vm and attached vlan 10 network adapter.

![](images/image-385.png)

![](images/image-386.png)

configured static ip address

![](images/image-387.png)

i have able to access the internet using pfsense network which we are using as vlans

![](images/image-388.png)

## Isolate Test Networks

Create isolated vSwitches within ESXi for test VMs, DMZ zones, or VPN-only networks, and let PFSense manage traffic between them — perfect for homelabs and security testing.
