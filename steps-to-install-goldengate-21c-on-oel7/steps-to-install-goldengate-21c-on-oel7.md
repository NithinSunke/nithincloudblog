---
title: "Steps to install Goldengate 21c on OEL7"
author: "Sunke Nithin"
date: 2024-06-22
categories: ["Uncategorized"]
tags: ["CLOUD", "DATABASE", "oci", "ORACLE", "oracle-database"]
source: https://nithinsunke.wordpress.com/2024/06/22/steps-to-install-goldengate-21c-on-oel7/
---

# Steps to install Goldengate 21c on OEL7

In this blog we will see how to install oracle Goldengate Microservice 21c on oel7.

#### Assumptions:

Oel7 os is already installed and downloaded gg 21c software from oracle support.

Download and copy the software to server

``` wp-block-code
[oracle@ogg u01]$ cd /u01/soft/
[oracle@ogg soft]$ ls
213000_fbo_ggs_Linux_x64_Oracle_services_shiphome.zip
```

Unzip the zip file

``` wp-block-code
[oracle@ogg soft]$ unzip 213000_fbo_ggs_Linux_x64_Oracle_services_shiphome.zip
Archive:  213000_fbo_ggs_Linux_x64_Oracle_services_shiphome.zip
   creating: fbo_ggs_Linux_x64_Oracle_services_shiphome/
   creating: fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/
   creating: fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/response/
.
.
.
  inflating: fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/runInstaller
  inflating: OGG-21.3.0.0-README.txt
  inflating: oracle-goldengate-release-notes_21.3.pdf
```

``` wp-block-code
[oracle@ogg soft]$ ls
213000_fbo_ggs_Linux_x64_Oracle_services_shiphome.zip  OGG-21.3.0.0-README.txt
fbo_ggs_Linux_x64_Oracle_services_shiphome             oracle-goldengate-release-notes_21.3.pdf

[oracle@ogg ~]$ cd /u01/soft/fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/
```

Install the goldengate using runInstaller

``` wp-block-code
[oracle@ogg Disk1]$ ./runInstaller
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 120 MB.   Actual 54806 MB    Passed
Checking swap space: must be greater than 150 MB.   Actual 4095 MB    Passed
Checking monitor: must be configured to display at least 256 colors.    Actual 16777216    Passed
Preparing to launch Oracle Universal Installer from /tmp/OraInstall2024-06-21_06-19-32AM. Please wait ...[oracle@ogg Disk1]$
```

![](images/image-9.png)


![](images/image-11.png)

![](images/image-12.png)

![](images/image-13.png)

![](images/image-14.png)

``` wp-block-code
[root@ogg ~]# /u01/app/oraInventory/orainstRoot.sh
Changing permissions of /u01/app/oraInventory.
Adding read,write permissions for group.
Removing read,write,execute permissions for world.

Changing groupname of /u01/app/oraInventory to oinstall.
The execution of the script is complete.
[root@ogg ~]#
```

``` wp-block-code
[oracle@ogg bin]$ pwd
/u01/app/ogg/gg_home/bin
[oracle@ogg bin]$ ./oggca.sh
```

![](images/image-15.png)

![](images/image-16.png)

![](images/image-17.png)

![](images/image-18.png)

![](images/image-19.png)

![](images/image-20.png)

![](images/image-21.png)

![](images/image-22.png)

![](images/image-23.png)

![](images/image-24.png)

![](images/image-25.png)

![](images/image-26.png)

``` wp-block-code
[root@ogg ~]# /u01/app/ogg/ogg_sm/bin/registerServiceManager.sh
Copyright (c) 2017, 2020, Oracle and/or its affiliates. All rights reserved.
----------------------------------------------------
     Oracle GoldenGate Install As Service Script
----------------------------------------------------
OGG_HOME=/u01/app/ogg/gg_home
OGG_CONF_HOME=/u01/app/ogg/ogg_sm/etc/conf
OGG_VAR_HOME=/u01/app/ogg/ogg_sm/var
OGG_USER=oracle
Running OracleGoldenGateInstall.sh...
Created symlink from /etc/systemd/system/multi-user.target.wants/OracleGoldenGate.service to /etc/systemd/system/OracleGoldenGate.service.
[root@ogg ~]#
```

![](images/image-27.png)

check the status of service

``` wp-block-code
[root@ogg ~]# systemctl status OracleGoldenGate.service
● OracleGoldenGate.service - Oracle GoldenGate Service Manager
   Loaded: loaded (/etc/systemd/system/OracleGoldenGate.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2024-06-21 06:39:24 UTC; 53s ago
 Main PID: 10048 (ServiceManager)
   CGroup: /system.slice/OracleGoldenGate.service
           └─10048 /u01/app/ogg/gg_home/bin/ServiceManager --inventory '/u01/app/ogg/ogg_sm/etc/conf'

Jun 21 06:39:24 ogg.scslabs.com systemd[1]: Started Oracle GoldenGate Service Manager.
Jun 21 06:39:24 ogg.scslabs.com ServiceManager[10048]: 2024-06-21T06:39:24.337+0000 INFO | Configuring user a...ed.
Jun 21 06:39:24 ogg.scslabs.com ServiceManager[10048]: 2024-06-21T06:39:24.426+0000 INFO | Configuring user a...e'.
Jun 21 06:39:24 ogg.scslabs.com ServiceManager[10048]: 2024-06-21T06:39:24.426+0000 WARN | Configuring user s...t'.
Jun 21 06:39:24 ogg.scslabs.com OracleGoldenGate.sh[10048]: Oracle GoldenGate Service Manager for Oracle
Jun 21 06:39:24 ogg.scslabs.com OracleGoldenGate.sh[10048]: Version 21.3.0.0.0 OGGCORE_21.3.0.0.0_PLATFORMS_2...047
Jun 21 06:39:24 ogg.scslabs.com OracleGoldenGate.sh[10048]: Copyright (C) 1995, 2021, Oracle and/or its affil...ed.
Jun 21 06:39:24 ogg.scslabs.com OracleGoldenGate.sh[10048]: Oracle Linux 7, x64, 64bit (optimized) on Jul 28 ...:40
Jun 21 06:39:24 ogg.scslabs.com OracleGoldenGate.sh[10048]: Operating system character set identified as UTF-8.
Hint: Some lines were ellipsized, use -l to show in full.
[root@ogg ~]#
```

![](images/image-28.png)

service manager console

![](images/image-29.png)

Admin Console

![](images/image-32.png)
