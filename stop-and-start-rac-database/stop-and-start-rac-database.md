---
title: "STOP and START RAC Database"
author: "Sunke Nithin"
date: 2019-07-04
categories: ["RAC"]
tags: []
source: https://nithinsunke.wordpress.com/2019/07/04/stop-and-start-rac-database/
---

# STOP and START RAC Database

**Stop RAC database:**

Source Grid profile as oracle user:

**.  ./grid_env   **

**(or)**

** . oraenv**

**ORACLE_SID = \[grid\] ?  +ASM1**

** **

**To Stop RAC database:**  

    srvctl stop database -d dbname -o immediate

  
 

**To check database status:**  

    srvctl status database -d dbname

  
**To Stop ASM:**  

    srvctl stop asm -n nodename1 –f

    srvctl stop asm -n nodename2 –f

  
 

**To Check ASM instance:**  

    srvctl status asm -n nodename1

    srvctl status asm -n nodename2

  
 

**To Stop Node apps:**  

    srvctl stop nodeapps -n nodename1 –f

    srvctl stop nodeapps -n nodename2 –f

  
 

**To Check Node apps status:**  

    srvctl status nodeapps -n nodename1

    srvctl status nodeapps -n nodename2

  
 

Login as **root** user and source grid profile

**To Stop Cluster:**  

    crsctl stop cluster –all

  
**To Check cluster:**  

    crsctl check cluster –all

  
 

**To Start RAC database:**

Login as **root** user and source

**. /home/grid/grid_env**

**(or)**

** . oraenv**

**ORACLE_SID = \[root\] ?  +ASM1**

 

**Start Cluster:**  

    crsctl start cluster –all ( Start cluster services on both nodes)

  
 

Login as **oracle** user and source . ./grid_env

**Start Database:**  

    srvctl start database –d RAC ( Start instances on both the nodes and start database)

  
 

**crs_stat -t –v** ( To Check the services running on both nodes)  

    crsctl stat res -t
