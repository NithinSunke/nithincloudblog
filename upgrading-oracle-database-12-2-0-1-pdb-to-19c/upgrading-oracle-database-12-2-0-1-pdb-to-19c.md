---
title: "Upgrading Oracle Database  12.2.0.1 PDB to 19c"
author: "Sunke Nithin"
date: 2024-05-21
categories: ["ORACLE", "UPGRADE"]
tags: ["DATABASE", "multitenant", "oracle-database"]
source: https://nithinsunke.wordpress.com/2024/05/21/upgrading-oracle-database-12-2-0-1-pdb-to-19c/
---

# Upgrading Oracle Database  12.2.0.1 PDB to 19c

In this blog we will see how to upgrade the pluggable database from 12.2.0.1 version to 19c version.

Assumptions:

1.  source and target vms are already built and ready for upgrade
2.  source db version is 12.2.0.1 and target db version is 19c
3.  An nfs share is configure between source and target for file copy.

## Check plug-in compatibility of the PDB in the new release CDB

We will start by checking whether my PDB can be plugged into the new release CDB.

![](images/image-21.png)

copy the file to the nfs share

![](images/image-22.png)

check plug-in compatibility on target system

![](images/image-23.png)

## Create a refreshable PDB on the new system

While I wait for downtime to start, I will create a refreshable PDB. I need to copy the PDB to the target system and to avoid doing that during downtime, I will use the refreshable PDB feature. When I refresh it, it will only need to apply the recent-most changes from the source PDB which is much faster than a full clone, obviously. I need a common user in the source CDB that can be used by the database link over which the cloning will take place:

## **on source database:**

![](images/image-24.png)

In the target CDB, create a database link that points to the source CDB:

![](images/image-25.png)

Check that it works

![](images/image-26.png)

create the refreshable PDB. we will set it to `REFRESH MODE MANUAL`.

![](images/image-27.png)

You can periodically refresh the PDB to further minimize the final refresh time:

![](images/image-28.png)

for testing create a sample test table in sys schema under O122V pdb on source.

![](images/image-29.png)

now on source close the pdb and do a final refresh of pdb at target

## source pdb:

![](images/image-30.png)

## on target:

do the final refresh:

![](images/image-31.png)

it is time to convert the refreshable PDB into a regular PDB

![](images/image-32.png)

Lets perform actual upgrade of pdb

open pdb in *upgrade* mode

![](images/image-33.png)

run dbupgrade to upgrade the pdb

![](images/image-35.png)

![](images/image-36.png)

open the pluggable database and validate the data

![](images/image-37.png)
