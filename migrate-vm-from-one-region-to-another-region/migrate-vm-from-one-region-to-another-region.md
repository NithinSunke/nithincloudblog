---
title: "migrate vm from one region to another region"
author: "Sunke Nithin"
date: 2025-01-09
categories: ["ORACLE CLOUD"]
tags: []
source: https://nithinsunke.wordpress.com/2025/01/09/migrate-vm-from-one-region-to-another-region/
---

# migrate vm from one region to another region

![](images/image-52.png)

**Migrating the Boot Volume and block vloume**

**Step 1: Create a Custom Image from your Compute Instance:**

From the Compute Instance Details page click on “Actions”, “Create Custom Image”.

![](images/image-53.png)

![](images/image-54.png)

Compute\> custom images

![](images/image-55.png)

![](images/image-56.png)

Setup replication of object storage bucket to target region in source region:

![](images/image-57.png)

![](images/image-58.png)

![](images/image-59.png)

Once exported

![](images/image-60.png)

See in target region object storage:

![](images/image-61.png)

Import to custom image:

![](images/image-62.png)

![](images/image-63.png)

![](images/image-64.png)

**Create a Compute Instance in the TARGET region:**

After the import completes, click on “Compute”, “Instances”, and “Create Instance”.

From the “Create Compute Instance” page, click on “Change Image Source”.

![](images/image-65.png)

![](images/image-66.png)

![](images/image-67.png)

![](images/image-68.png)

![](images/image-69.png)

![](images/image-70.png)

![](images/image-71.png)

![](images/image-72.png)

![](images/image-73.png)

![](images/image-74.png)

Create bastion service and connect to the server

![](images/image-75.png)

The new created Compute Instance now has exactly the same Boot Volume as the source Compute VM, i.e. it has the data and software installed on the Boot Volume.

**Migrating the Block Volumes**

In case you are using additional Block Volumes on you source Compute VM, we can migrate them as follows

**Take a backup of your Block Volume**

From the Compute Instance Details page, click on “Attached Block Volumes”, then on the menu on the right side, and on “View Block Volume Details”

On source:

![](images/image-76.png)

Click on disk name

![](images/image-77.png)

![](images/image-78.png)

![](images/image-79.png)

![](images/image-80.png)

Now copy the backup of blockvolume to target region

![](images/image-81.png)

![](images/image-82.png)

Check the process in target region:

![](images/image-83.png)

Now backup is available in target region:

![](images/image-84.png)

**Create a new Block Volume in the TARGET region**

![](images/image-85.png)

![](images/image-86.png)

![](images/image-87.png)

![](images/image-88.png)

**Attach the Block Volume to the Compute Instance in the TARGET region**

Now on the compute instance page attach the block volume to the instance.

![](images/image-89.png)

![](images/image-90.png)

![](images/image-91.png)

![](images/image-92.png)

Login to server and verify

![](images/image-93.png)
