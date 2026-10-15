---
title: "install oracle database 11.2.0.4 steps"
author: "Sunke Nithin"
date: 2019-06-13
categories: ["DATABASE"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/13/install-oracle-database-11-2-0-4-steps/
---

# install oracle database 11.2.0.4 steps

Hi guys today i will show how to install oracle database 11.2.0.4 on oracle linux 7 database

1)insall oracle linux 7  
use below document to intall oracle linux  
<https://drive.google.com/open?id=15jIwpPBKNsjsRuEaaTuJUrrSCrSHIck1>

2\) download oracle database software from support.oracle.com  

    patch no : 13390677

    p13390677_112040_Linux-x86-64_1of7.zip

    p13390677_112040_Linux-x86-64_2of7.zip

    p13390677_112040_Linux-x86-64_3of7.zip

    p13390677_112040_Linux-x86-64_5of7.zip

    p13390677_112040_Linux-x86-64_6of7.zip

  
3)unzip the oracle software  

    unzip '*'

  
4) install oracle database rpm  

    oracle-rdbms-server-11gR2-preinstall.x86_64

  

    [root@oradbdvl01 yum.repos.d]# yum install oracle-rdbms-server-11gR2-preinstall.x86_64

    Loaded plugins: langpacks, ulninfo

    Resolving Dependencies

    --> Running transaction check

    ---> Package oracle-rdbms-server-11gR2-preinstall.x86_64 0:1.0-6.el7 will be installed

    --> Processing Dependency: gcc for package: oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64

    --> Processing Dependency: gcc-c++ for package: oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64

    --> Processing Dependency: ksh for package: oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64

    --> Processing Dependency: libaio-devel for package: oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64

    --> Processing Dependency: glibc-devel for package: oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64

    --> Processing Dependency: libstdc++-devel for package: oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64

    --> Running transaction check

    ---> Package gcc.x86_64 0:4.8.5-36.0.1.el7_6.2 will be installed

    --> Processing Dependency: cpp = 4.8.5-36.0.1.el7_6.2 for package: gcc-4.8.5-36.0.1.el7_6.2.x86_64

    --> Processing Dependency: libgomp = 4.8.5-36.0.1.el7_6.2 for package: gcc-4.8.5-36.0.1.el7_6.2.x86_64

    --> Processing Dependency: libgcc >= 4.8.5-36.0.1.el7_6.2 for package: gcc-4.8.5-36.0.1.el7_6.2.x86_64

    ---> Package gcc-c++.x86_64 0:4.8.5-36.0.1.el7_6.2 will be installed

    --> Processing Dependency: libstdc++ = 4.8.5-36.0.1.el7_6.2 for package: gcc-c++-4.8.5-36.0.1.el7_6.2.x86_64

    ---> Package glibc-devel.x86_64 0:2.17-260.0.17.el7_6.6 will be installed

    --> Processing Dependency: glibc = 2.17-260.0.17.el7_6.6 for package: glibc-devel-2.17-260.0.17.el7_6.6.x86_64

    --> Processing Dependency: glibc-headers = 2.17-260.0.17.el7_6.6 for package: glibc-devel-2.17-260.0.17.el7_6.6.x86_64

    --> Processing Dependency: glibc-headers for package: glibc-devel-2.17-260.0.17.el7_6.6.x86_64

    ---> Package ksh.x86_64 0:20120801-139.0.1.el7 will be installed

    ---> Package libaio-devel.x86_64 0:0.3.109-13.el7 will be installed

    ---> Package libstdc++-devel.x86_64 0:4.8.5-36.0.1.el7_6.2 will be installed

    --> Running transaction check

    ---> Package cpp.x86_64 0:4.8.5-36.0.1.el7 will be updated

    ---> Package cpp.x86_64 0:4.8.5-36.0.1.el7_6.2 will be an update

    ---> Package glibc.x86_64 0:2.17-260.0.9.el7 will be updated

    --> Processing Dependency: glibc = 2.17-260.0.9.el7 for package: glibc-common-2.17-260.0.9.el7.x86_64

    ---> Package glibc.x86_64 0:2.17-260.0.17.el7_6.6 will be an update

    ---> Package glibc-headers.x86_64 0:2.17-260.0.17.el7_6.6 will be installed

    ---> Package libgcc.x86_64 0:4.8.5-36.0.1.el7 will be updated

    ---> Package libgcc.x86_64 0:4.8.5-36.0.1.el7_6.2 will be an update

    ---> Package libgomp.x86_64 0:4.8.5-36.0.1.el7 will be updated

    ---> Package libgomp.x86_64 0:4.8.5-36.0.1.el7_6.2 will be an update

    ---> Package libstdc++.x86_64 0:4.8.5-36.0.1.el7 will be updated

    ---> Package libstdc++.x86_64 0:4.8.5-36.0.1.el7_6.2 will be an update

    --> Running transaction check

    ---> Package glibc-common.x86_64 0:2.17-260.0.9.el7 will be updated

    ---> Package glibc-common.x86_64 0:2.17-260.0.17.el7_6.6 will be an update

    --> Finished Dependency Resolution

    Dependencies Resolved

    =====================================================================================================================================================================

    Package Arch Version Repository Size

    =====================================================================================================================================================================

    Installing:

    oracle-rdbms-server-11gR2-preinstall x86_64 1.0-6.el7 ol7_latest 22 k

    Installing for dependencies:

    gcc x86_64 4.8.5-36.0.1.el7_6.2 ol7_latest 16 M

    gcc-c++ x86_64 4.8.5-36.0.1.el7_6.2 ol7_latest 7.2 M

    glibc-devel x86_64 2.17-260.0.17.el7_6.6 ol7_latest 1.1 M

    glibc-headers x86_64 2.17-260.0.17.el7_6.6 ol7_latest 685 k

    ksh x86_64 20120801-139.0.1.el7 ol7_latest 883 k

    libaio-devel x86_64 0.3.109-13.el7 ol7_latest 12 k

    libstdc++-devel x86_64 4.8.5-36.0.1.el7_6.2 ol7_latest 1.5 M

    Updating for dependencies:

    cpp x86_64 4.8.5-36.0.1.el7_6.2 ol7_latest 5.9 M

    glibc x86_64 2.17-260.0.17.el7_6.6 ol7_latest 3.7 M

    glibc-common x86_64 2.17-260.0.17.el7_6.6 ol7_latest 12 M

    libgcc x86_64 4.8.5-36.0.1.el7_6.2 ol7_latest 102 k

    libgomp x86_64 4.8.5-36.0.1.el7_6.2 ol7_latest 157 k

    libstdc++ x86_64 4.8.5-36.0.1.el7_6.2 ol7_latest 304 k

    Transaction Summary

    =====================================================================================================================================================================

    Install 1 Package (+7 Dependent packages)

    Upgrade ( 6 Dependent packages)

    Total download size: 49 M

    Is this ok [y/d/N]: y

    Downloading packages:

    No Presto metadata available for ol7_latest

    (1/14): cpp-4.8.5-36.0.1.el7_6.2.x86_64.rpm | 5.9 MB 00:00:02

    (2/14): gcc-c++-4.8.5-36.0.1.el7_6.2.x86_64.rpm | 7.2 MB 00:00:03

    (3/14): glibc-2.17-260.0.17.el7_6.6.x86_64.rpm | 3.7 MB 00:00:01

    (4/14): glibc-common-2.17-260.0.17.el7_6.6.x86_64.rpm | 12 MB 00:00:05

    (5/14): glibc-devel-2.17-260.0.17.el7_6.6.x86_64.rpm | 1.1 MB 00:00:00

    (6/14): glibc-headers-2.17-260.0.17.el7_6.6.x86_64.rpm | 685 kB 00:00:00

    (7/14): ksh-20120801-139.0.1.el7.x86_64.rpm | 883 kB 00:00:00

    (8/14): libaio-devel-0.3.109-13.el7.x86_64.rpm | 12 kB 00:00:00

    (9/14): libgcc-4.8.5-36.0.1.el7_6.2.x86_64.rpm | 102 kB 00:00:00

    (10/14): libgomp-4.8.5-36.0.1.el7_6.2.x86_64.rpm | 157 kB 00:00:00

    (11/14): libstdc++-4.8.5-36.0.1.el7_6.2.x86_64.rpm | 304 kB 00:00:00

    (12/14): libstdc++-devel-4.8.5-36.0.1.el7_6.2.x86_64.rpm | 1.5 MB 00:00:00

    (13/14): oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64.rpm | 22 kB 00:00:00

    (14/14): gcc-4.8.5-36.0.1.el7_6.2.x86_64.rpm | 16 MB 00:00:17

    ---------------------------------------------------------------------------------------------------------------------------------------------------------------------

    Total 2.9 MB/s | 49 MB 00:00:17

    Running transaction check

    Running transaction test

    Transaction test succeeded

    Running transaction

    Updating : libgcc-4.8.5-36.0.1.el7_6.2.x86_64 1/20

    Updating : glibc-common-2.17-260.0.17.el7_6.6.x86_64 2/20

    Updating : glibc-2.17-260.0.17.el7_6.6.x86_64 3/20

    Updating : libstdc++-4.8.5-36.0.1.el7_6.2.x86_64 4/20

    Installing : libstdc++-devel-4.8.5-36.0.1.el7_6.2.x86_64 5/20

    Installing : ksh-20120801-139.0.1.el7.x86_64 6/20

    Updating : libgomp-4.8.5-36.0.1.el7_6.2.x86_64 7/20

    Installing : glibc-headers-2.17-260.0.17.el7_6.6.x86_64 8/20

    Installing : glibc-devel-2.17-260.0.17.el7_6.6.x86_64 9/20

    Updating : cpp-4.8.5-36.0.1.el7_6.2.x86_64 10/20

    Installing : gcc-4.8.5-36.0.1.el7_6.2.x86_64 11/20

    Installing : gcc-c++-4.8.5-36.0.1.el7_6.2.x86_64 12/20

    Installing : libaio-devel-0.3.109-13.el7.x86_64 13/20

    Installing : oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64 14/20

    Cleanup : libstdc++-4.8.5-36.0.1.el7.x86_64 15/20

    Cleanup : cpp-4.8.5-36.0.1.el7.x86_64 16/20

    Cleanup : libgomp-4.8.5-36.0.1.el7.x86_64 17/20

    Cleanup : glibc-common-2.17-260.0.9.el7.x86_64 18/20

    Cleanup : glibc-2.17-260.0.9.el7.x86_64 19/20

    Cleanup : libgcc-4.8.5-36.0.1.el7.x86_64 20/20

    Verifying : ksh-20120801-139.0.1.el7.x86_64 1/20

    Verifying : glibc-2.17-260.0.17.el7_6.6.x86_64 2/20

    Verifying : oracle-rdbms-server-11gR2-preinstall-1.0-6.el7.x86_64 3/20

    Verifying : libgcc-4.8.5-36.0.1.el7_6.2.x86_64 4/20

    Verifying : libgomp-4.8.5-36.0.1.el7_6.2.x86_64 5/20

    Verifying : libstdc++-devel-4.8.5-36.0.1.el7_6.2.x86_64 6/20

    Verifying : gcc-4.8.5-36.0.1.el7_6.2.x86_64 7/20

    Verifying : glibc-headers-2.17-260.0.17.el7_6.6.x86_64 8/20

    Verifying : cpp-4.8.5-36.0.1.el7_6.2.x86_64 9/20

    Verifying : glibc-devel-2.17-260.0.17.el7_6.6.x86_64 10/20

    Verifying : libaio-devel-0.3.109-13.el7.x86_64 11/20

    Verifying : glibc-common-2.17-260.0.17.el7_6.6.x86_64 12/20

    Verifying : libstdc++-4.8.5-36.0.1.el7_6.2.x86_64 13/20

    Verifying : gcc-c++-4.8.5-36.0.1.el7_6.2.x86_64 14/20

    Verifying : libstdc++-4.8.5-36.0.1.el7.x86_64 15/20

    Verifying : glibc-common-2.17-260.0.9.el7.x86_64 16/20

    Verifying : cpp-4.8.5-36.0.1.el7.x86_64 17/20

    Verifying : libgomp-4.8.5-36.0.1.el7.x86_64 18/20

    Verifying : libgcc-4.8.5-36.0.1.el7.x86_64 19/20

    Verifying : glibc-2.17-260.0.9.el7.x86_64 20/20

    Installed:

    oracle-rdbms-server-11gR2-preinstall.x86_64 0:1.0-6.el7

    Dependency Installed:

    gcc.x86_64 0:4.8.5-36.0.1.el7_6.2 gcc-c++.x86_64 0:4.8.5-36.0.1.el7_6.2 glibc-devel.x86_64 0:2.17-260.0.17.el7_6.6 glibc-headers.x86_64 0:2.17-260.0.17.el7_6.6

    ksh.x86_64 0:20120801-139.0.1.el7 libaio-devel.x86_64 0:0.3.109-13.el7 libstdc++-devel.x86_64 0:4.8.5-36.0.1.el7_6.2

    Dependency Updated:

    cpp.x86_64 0:4.8.5-36.0.1.el7_6.2 glibc.x86_64 0:2.17-260.0.17.el7_6.6 glibc-common.x86_64 0:2.17-260.0.17.el7_6.6 libgcc.x86_64 0:4.8.5-36.0.1.el7_6.2

    libgomp.x86_64 0:4.8.5-36.0.1.el7_6.2 libstdc++.x86_64 0:4.8.5-36.0.1.el7_6.2

    Complete!

  
5) disable selinux  

    vi /etc/selinux/config

    SELINUX=disabled

  
6) add hostname entrie to /etc/hosts  

    [root@oradbdvl01 yum.repos.d]# vi /etc/hosts

    127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4

    ::1 localhost localhost.localdomain localhost6 localhost6.localdomain6

    192.168.58.2 openfiler.snv.ind openfiler

  
7) create groups and users

preinstall rpm will take care of groups and users  

    [root@oradbdvl01 yum.repos.d]# id oracle

    uid=54321(oracle) gid=54321(oinstall) groups=54321(oinstall),54322(dba)

  
8) give persmissions on software directory to oracle users  

    cd /soft/db/

    chown -R oracle:dba database examples

    chown -R oracle:dba /a01 /b01 /c01 /d01

  
9) enable password to oracle user  

    passwd oracle

  
10) login to new sesssion as oracle user  
11) run ./runInstaller  

    i)uncheck the check box and next

    ii) you have not provided an email address. yes

    iii)skip software updates next

    iv) install database software only

    v) single instance database

    vi) select language english

    vii) enterprise edition

    viii) specify installation location

    xi) create central inventory

    x) privileged os groups

    xi) preprquisite check

    if any checks failed need to work on that before continue

    xii) summary

    xiii) installation process

  
after installation at end it ask to run two scripts as root user  

    [root@oradbdvl01 /]# /a01/oraInventory/orainstRoot.sh

    Changing permissions of /a01/oraInventory.

    Adding read,write permissions for group.

    Removing read,write,execute permissions for world.

    Changing groupname of /a01/oraInventory to oinstall.

    The execution of the script is complete.

  

    [root@oradbdvl01 /]# /a01/oracle/11.2.0/db/root.sh

    Performing root user operation for Oracle 11g

    The following environment variables are set as:

    ORACLE_OWNER= oracle

    ORACLE_HOME= /a01/oracle/11.2.0/db

    Enter the full pathname of the local bin directory: [/usr/local/bin]:

    Copying dbhome to /usr/local/bin ...

    Copying oraenv to /usr/local/bin ...

    Copying coraenv to /usr/local/bin ...

    Creating /etc/oratab file...

    Entries will be added to the /etc/oratab file as needed by

    Database Configuration Assistant when a database is created

    Finished running generic part of root script.

    Now product-specific root actions will be performed.

    Finished product-specific root actions.

    [root@oradbdvl01 /]#
