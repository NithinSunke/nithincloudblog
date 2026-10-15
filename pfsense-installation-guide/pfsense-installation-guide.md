---
title: "PFSense Installation Guide"
author: "Sunke Nithin"
date: 2025-04-16
categories: ["FIREWALL", "LAB"]
tags: []
source: https://nithinsunke.wordpress.com/2025/04/16/pfsense-installation-guide/
---

# PFSense Installation Guide

**PFSense** is a powerful, open-source firewall and router platform based on FreeBSD. It’s widely used for home labs, small businesses, and enterprise networks for routing, VPN, DHCP, DNS, and more.

In this post, I’ll walk you through the step-by-step process of installing **PFSense**.

## 📦 Prerequisites

Before you start:

- A system or VM with at least:
  - 1 GB RAM (2 GB+ recommended)
  - 8 GB disk space
  - 2 network interfaces (WAN & LAN)
- PFSense ISO image — download from https://www.pfsense.org/download/
- A bootable USB (for physical installation) or ISO attached to your VM

## 📌 Step-by-Step Installation

### 📥 1️⃣ Download PFSense ISO

- Visit https://www.pfsense.org/download/
- Choose:
  - **Architecture**: AMD64 (64-bit)
  - **Installer**: ISO installer
  - **Mirror**: Choose the nearest one

![](images/image-310.png)

![](images/image-312.png)

![](images/image-314.png)

in vmware workstation create 2 host-only networks.

select virtual network editor

![](images/image-316.png)

![](images/image-317.png)

![](images/image-318.png)

![](images/image-319.png)

![](images/image-320.png)

![](images/image-321.png)

![](images/image-322.png)

rename the network adapters

![](images/image-323.png)

![](images/image-324.png)

for each adapter provide the static ip

![](images/image-325.png)

![](images/image-326.png)

create pfsense virtual machine

![](images/image-327.png)

add the host-only network adapters

![](images/image-328.png)

start the vm

![](images/image-329.png)

![](images/image-330.png)

![](images/image-331.png)

![](images/image-332.png)

![](images/image-333.png)

![](images/image-334.png)

![](images/image-335.png)

![](images/image-336.png)

![](images/image-337.png)

![](images/image-338.png)

![](images/image-339.png)

once restarted you will see the login screen

![](images/image-340.png)

configure the interfaces

![](images/image-341.png)

![](images/image-342.png)

login to pfsense console

![](images/image-343.png)

default credentials: admin/pfsense

once you login for first time it will ask for pfsense setup

![](images/image-345.png)

![](images/image-346.png)

![](images/image-347.png)

![](images/image-348.png)

![](images/image-349.png)

![](images/image-350.png)

![](images/image-351.png)

![](images/image-352.png)

![](images/image-353.png)

once the setup is done it will reload the configuration

![](images/image-354.png)

now assign the interfaces

![](images/image-355.png)

![](images/image-356.png)

![](images/image-357.png)

![](images/image-358.png)

![](images/image-359.png)

![](images/image-360.png)

![](images/image-361.png)

![](images/image-362.png)

![](images/image-363.png)

![](images/image-364.png)

steps to configure firewall rules

![](images/image-365.png)

![](images/image-366.png)

![](images/image-367.png)

![](images/image-368.png)

![](images/image-369.png)

![](images/image-370.png)

![](images/image-371.png)

![](images/image-372.png)
