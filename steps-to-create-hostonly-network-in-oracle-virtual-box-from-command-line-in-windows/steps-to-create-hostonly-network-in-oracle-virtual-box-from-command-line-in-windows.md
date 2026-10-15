---
title: "steps to create hostonly network in oracle virtual box from command line in windows"
author: "Sunke Nithin"
date: 2024-05-24
categories: ["Automation Tools", "MY LAB SETUP"]
tags: []
source: https://nithinsunke.wordpress.com/2024/05/24/steps-to-create-hostonly-network-in-oracle-virtual-box-from-command-line-in-windows/
---

# steps to create hostonly network in oracle virtual box from command line in windows

Below are the steps to create host only network from command line on windows servers

1.  to create a hostonly network

VBoxManage hostonlyif create

![](images/image-38.png)

2\. list the available hostonly networks on host

VBoxManage list -l hostonlyifs

![](images/image-39.png)

3\. Modify the values as required

VBoxManage hostonlyif ipconfig "VirtualBox Host-Only Ethernet Adapter" --ip 172.16.0.1 --netmask 255.255.0.0

![](images/image-40.png)

![](images/image-41.png)
