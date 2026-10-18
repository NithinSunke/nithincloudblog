---
title: "clone of linux vm instance"
author: "Sunke Nithin"
date: 2025-01-09
categories: ["ORACLE CLOUD", "Uncategorized"]
tags: []
source: https://nithinsunke.wordpress.com/2025/01/09/clone-of-linux-vm-instance/
---

# clone of linux vm instance

**source vm:**

![](images/image-2.png)

**For demo**

Install https server and create a web page

![](images/image-3.png)

![](images/image-4.png)

![](images/image-5.png)

Now open the port at security list

![](images/image-6.png)

Open the firewalld at server level

![](images/image-7.png)

Open the browser and give ip address of the web server.

![](images/image-8.png)

Create a custom index page.

![](images/image-9.png)

Reload the page

![](images/image-10.png)

Now we will clone the vm

Stop the vm

![](images/image-11.png)

Vm is stopped

![](images/image-12.png)

Three Ways to Clone your Compute Instance in Oracle Cloud Infrastructure

1.  Cloning
2.  Backup and restore
3.  Custom image

**Cloning:**

Create a volume group for easy cloning process and to have time consistent copy of all disks

![](images/image-13.png)

![](images/image-14.png)

Create volume group

![](images/image-15.png)

![](images/image-16.png)

![](images/image-17.png)

![](images/image-18.png)

![](images/image-19.png)

![](images/image-20.png)

![](images/image-21.png)

![](images/image-23.png)

![](images/image-24.png)

![](images/image-25.png)

Select the boot volume

![](images/image-26.png)

Create new instance from clone volume group

![](images/image-27.png)

![](images/image-28.png)

![](images/image-29.png)

![](images/image-30.png)

![](images/image-31.png)

![](images/image-32.png)

![](images/image-33.png)

![](images/image-34.png)

**Backup and restore:**

1.  Create a volume group and add boot volume for time consistent of all disks

![](images/image-35.png)

Backup the volume group. The Backup will reside on Object Storage.

![](images/image-36.png)

![](images/image-37.png)

![](images/image-38.png)

![](images/image-39.png)

![](images/image-40.png)

![](images/image-41.png)

![](images/image-42.png)

![](images/image-43.png)

![](images/image-44.png)

![](images/image-45.png)

Create instance

You can use same method as above to create instance

1.  Clone using custom image

![](images/image-46.png)

![](images/image-47.png)

![](images/image-48.png)

![](images/image-49.png)

Create instance using custom image

![](images/image-50.png)

![](images/image-51.png)
