---
title: "How to Install VMware ESXi 7: Step-by-Step Guide for Beginners"
author: "Sunke Nithin"
date: 2025-05-04
categories: ["Virtualization"]
tags: ["CLOUD", "Linux", "technology", "vmware", "WINDOWS"]
source: https://nithinsunke.wordpress.com/2025/05/04/how-to-install-vmware-esxi-7-step-by-step-guide-for-beginners/
---

# How to Install VMware ESXi 7: Step-by-Step Guide for Beginners

If you're building your own homelab or looking to virtualize your IT infrastructure, **VMware ESXi 7** is a great hypervisor to start with. It's lightweight, reliable, and widely used in both enterprise and lab environments. In this guide, I’ll walk you through the complete process of installing **VMware ESXi 7** on a physical server or compatible hardware.

## Prerequisites

Before you begin, ensure you have:

- A physical server or PC with **VT-x/AMD-V enabled in BIOS**
- At least **4GB RAM** and a **dual-core CPU**
- Minimum **8GB USB drive** (for installation media)
- **VMware ESXi 7 ISO** (Download from VMware official site)
- Access to another computer for creating the bootable installer
- Optional: Static IP, DNS settings, and hostname

## Step 1: Create a Bootable USB Installer

1.  **Download the ESXi 7 ISO** file from the VMware website.
2.  Use a tool like **Rufus** (on Windows) or **Balena Etcher** (cross-platform) to create a bootable USB drive.
3.  Select the downloaded ISO and USB drive, and start the process.

## Step 2: Boot from the USB and Start Installation

1.  Plug the USB into your server and boot into BIOS.
2.  Set USB as the **first boot device**.
3.  Save and reboot — the system will boot into the ESXi installer.

![](images/image.png)

![](images/image-1.png)

## Step 3: Install ESXi 7

On the welcome screen, press **Enter** to begin installation.

![](images/image-2.png)

Accept the **EULA** and continue.

![](images/image-6.png)

Select the **disk** where you want to install ESXi (e.g., SSD or HDD).

![](images/image-8.png)

Choose the keyboard layout and set a **root password**.

![](images/image-11.png)

![](images/image-12.png)

Confirm and proceed — installation takes a few minutes.

![](images/image-14.png)

![](images/image-16.png)

![](images/image-18.png)

Once complete, remove the USB and reboot the system.

![](images/image-22.png)

## 🌐 Step 4: Configure ESXi from DCUI (Direct Console User Interface)

1.  After reboot, you'll land on the **yellow-gray DCUI screen**.
2.  Press **F2** to log in with root credentials.
3.  Set the following:
    - **Static IP address**
    - **Subnet mask**
    - **Gateway and DNS**
    - **Hostname**
4.  Save and restart network management if prompted.

![](images/image-24.png)

![](images/image-25.png)

![](images/image-28.png)

![](images/image-29.png)

![](images/image-30.png)

![](images/image-31.png)

![](images/image-32.png)

## 💻 Step 5: Access ESXi via Web Interface

1.  From a browser on your local network, go to:  
    `https://<your-ESXi-IP-address>`
2.  Accept the SSL warning and log in using the **root credentials**.
3.  You're now in the **vSphere Host Client**, ready to create and manage virtual machines!

https://192.168.1.100

![](images/image-33.png)

![](images/image-34.png)

![](images/image-35.png)

#### Optional: Post-Installation Tips

If using license, apply it via the web UI → *Manage → Licensing*.

![](images/image-37.png)

![](images/image-38.png)

![](images/image-39.png)

### create a new storage for vm's

![](images/image-40.png)

![](images/image-41.png)

![](images/image-42.png)

![](images/image-43.png)

![](images/image-44.png)

![](images/image-45.png)

![](images/image-46.png)

## You're Done!

You've now successfully installed **VMware ESXi 7** and are ready to start building virtual machines or even a nested virtualization lab. ESXi is a powerful foundation for your home or enterprise virtual environment.

Let me know in the comments if you'd like a follow-up guide on installing vCenter, creating VMs.
