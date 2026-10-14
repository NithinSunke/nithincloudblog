---
title: "Task1-Install Windows Server on VM or physical machine"
author: "Sunke Nithin"
date: 2025-05-06
categories: ["WINDOWS"]
tags: []
source: https://nithinsunke.wordpress.com/2025/05/06/task1-install-windows-server-on-vm-or-physical-machine/
---

# Task1-Install Windows Server on VM or physical machine

Installing **Windows Server 2022** on **Proxmox VE** gives you the best of both worlds: a modern Windows OS and a powerful open-source hypervisor. Whether you’re setting up a homelab or evaluating enterprise solutions, this guide walks you through the full process.

** What You’ll Need**

Before you begin, make sure you have the following:

- Proxmox VE (v6.4 or newer recommended)
- Windows Server 2022 ISO (Evaluation or Licensed)
- VirtIO Drivers ISO (for Windows to recognize Proxmox hardware)  
  [Download here](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.271-1/virtio-win.iso)

**Step-by-Step Installation**

**Step 1: Upload ISO Files to Proxmox**

![](images/image-133.png)

**Step 2: Create the Virtual Machine**

![](images/image-119.png)

![](images/image-105.png)

![](images/image-108.png)

![](images/image-104.png)

![](images/image-110.png)

![](images/image-115.png)

![](images/image-114.png)

![](images/image-109.png)

![](images/image-111.png)

![](images/image-113.png)

Add another cd/dvd for virtio drivers iso.

![](images/image-117.png)

![](images/image-112.png)

**Step 4: Start the VM & Begin Installation**

![](images/image-103.png)

![](images/image-123.png)

![](images/image-107.png)

![](images/image-132.png)

![](images/image-116.png)

![](images/image-130.png)

![](images/image-122.png)

Load driver from iso

![](images/image-129.png)

![](images/image-127.png)

![](images/image-121.png)

![](images/image-126.png)

![](images/image-125.png)

![](images/image-120.png)

![](images/image-128.png)

![](images/image-118.png)

![](images/image-124.png)

![](images/image-131.png)

**Step 6: Install the QEMU Guest Agent**

1.  Mount the VirtIO ISO inside the guest.
2.  Navigate to:  
      
    CopyEdit  
    guest-agent \> qemu-ga-x86_64.msi
3.  Install it — this enables graceful shutdown, IP reporting, and better integration with Proxmox.

**Final Thoughts**

Running Windows Server on Proxmox is a great way to combine the reliability of Linux with the functionality of Windows. Whether you're running a lab, test environment, or production workload, this setup gives you flexibility, performance, and control.
