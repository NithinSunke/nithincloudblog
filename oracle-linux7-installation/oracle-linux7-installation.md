---
title: "Oracle Linux7 Installation"
author: "Sunke Nithin"
date: 2019-06-13
categories: ["Linux"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/13/oracle-linux7-installation/
---

# Oracle Linux7 Installation

We are going to install Oracle Linux 7 on VMware Workstation 15. Oracle Linux, formerly known as Oracle Enterprise Linux.

You can download Oracle Linux through Oracle's E-delivery service or from a variety of mirror sites.

Lets Start with Installation..................................  
Download latest Oracle Linux release from Oracle E-delivery. Current release is Oracle Linux 7 Update-1.

Open VMware Workstation 12 -\> File -\> New Virtual Machine-\> Typical  
Click Next

Select Installer DISC image file and browse the path to Oracle Linux 7 .ISO. and Click Next

Enter the name for your Virtual Machine and browse the location where you want to save the Virtual Machine. Click Next

Set the maximum disk size and you can chose either split the virtual disk to store as single file. Click Next

Click Finish

Click on Edit Virtual Machine settings.

Set the Memory to 2GB, Processors as per your hardware recommendations and Network to bridged.

Power on the virtual machine. Select Install Oracle Linux 7.1 and hit Enter.

Select you preferred language and click Continue

Set your date and time zone, Set your Installation media i.e Local Media and click Software selection. Select Server with GUI and select following Add-Ons for Selected environment

1\. Java Platform  
2. KDE  
3. Compatibility Libraries

You can chose what ever you want in your environment. I am choosing what ever i need in my environment. Click Done.

Now Click Installation Destination. Click I will configure partitioning and click Done

Select Standard Partition and click + button to dd mount point. I am going to set Mount Point / with capacity 37500 Click Add mount point.

Set the File system to ext4

Add Swap Mount point for the rest of disk space and Click Done.

Accept changes.

Click Begin Installation.

Set the root password on next screen.

Create a User by clicking USER CREATION. Click done.

You can see Installation Progress. It will take some time to complete.

Click Reboot to finish the Installation.

Accept the licensing and Click Done. Click Finish Configuration.

Select No, I prefer to register at a later time. Click Forward, Click No Thanks I'll Connect later and Click Forward

Enter your password to Sign In. Click Sign In

Click Next, Next

Click Start using Oracle Linux Server

You will see Internet is working if you are connected to a rite bridged network on your physical machine.

That's all
