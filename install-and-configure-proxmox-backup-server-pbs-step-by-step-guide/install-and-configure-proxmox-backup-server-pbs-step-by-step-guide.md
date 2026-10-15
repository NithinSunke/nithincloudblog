---
title: "Install and Configure Proxmox Backup Server (PBS) — Step-by-Step Guide"
author: "Sunke Nithin"
date: 2025-04-22
categories: ["Virtualization"]
tags: []
source: https://nithinsunke.wordpress.com/2025/04/22/install-and-configure-proxmox-backup-server-pbs-step-by-step-guide/
---

# Install and Configure Proxmox Backup Server (PBS) — Step-by-Step Guide

**Proxmox Backup Server (PBS)** is an enterprise-grade backup solution for Proxmox VE environments. It provides efficient, secure, and incremental backups for VMs, containers, and physical hosts.

This guide walks you through installing and configuring PBS as a VM on Proxmox VE.

## Prerequisites

- A running **Proxmox VE server**
- **Proxmox Backup Server ISO**  
  Download from: https://www.proxmox.com/en/downloads
- At least **2 vCPUs, 2 GB RAM, and 32 GB disk space** for the PBS VM

## Step 1: Welcome screen

install proxmox backup server (graphical)

![](images/image-480.png)

![](images/image-482.png)

Follow the installation wizard:

- Agree to license

![](images/image-484.png)

- Select target hard disk

![](images/image-486.png)

- Configure root password and management interface IP address

![](images/image-488.png)

![](images/image-490.png)

![](images/image-492.png)

![](images/image-494.png)

![](images/image-496.png)

Finish installation and reboot the VM

## Step 2: Access Proxmox Backup Server Web UI

- Open your browser
- Go to:  
  `https://<PBS_VM_IP>:8007`
- Login with **root** and your configured password

![](images/image-498.png)

## Step 3: Configure Data Storage in PBS

1.  In the PBS Web UI:
    - Go to **Datastore**
    - Click **Create Datastore**
    - Name your datastore (e.g., `proxmox-backups`)
    - Select storage path (can be a mounted disk or directory)
    - Save

![](images/image-500.png)

## Step 4: Add Proxmox VE to PBS

1.  In **Proxmox VE Web GUI**:
    - Go to **Datacenter → Storage → Add → Proxmox Backup Server**
    - Enter:
      - **ID**: pbs-backup
      - **Server**: PBS VM IP
      - **Username**: root@pam
      - **Password**: (PBS root password)
      - **Datastore**: Select the one you created
    - Save

![](images/image-502.png)

![](images/image-504.png)

![](images/image-506.png)

![](images/image-509.png)

![](images/image-508.png)

![](images/image-510.png)

![](images/image-511.png)

## 🎉 Done!

Now your **Proxmox VE** can back up VMs and containers to your **Proxmox Backup Server VM** efficiently, with incremental deduplicated backups, and fast restores.

## Bonus Tips:

- Schedule backups using **Proxmox VE Datacenter → Backup**
- Use **Prune & Verify Jobs** in PBS to manage storage usage
- Monitor backup performance via PBS dashboard
