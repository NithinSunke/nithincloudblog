---
title: "Hostname changed on grid infrastructure is running."
author: "Sunke Nithin"
date: 2020-02-06
categories: ["RAC"]
tags: []
source: https://nithinsunke.wordpress.com/2020/02/06/hostname-changed-on-grid-infrastructure-is-running/
---

# Hostname changed on grid infrastructure is running.

1\) before changing the hostname check the status of resources  
\[root@asmdvl01 ~\]# . oraenv  
ORACLE_SID = \[root\] ? +ASM  
The Oracle base has been set to /a01/app/grid_base  

    [root@asmdvl01 ~]# crsctl stat res -t

    --------------------------------------------------------------------------

    Name Target State Server State details

    --------------------------------------------------------------------------

    Local Resources

    --------------------------------------------------------------------------

    ora.CRS.dg                  ONLINE ONLINE asmdvl01 STABLE

    ora.DATA.dg                 ONLINE ONLINE asmdvl01 STABLE

    ora.FRA.dg                  ONLINE ONLINE asmdvl01 STABLE

    ora.LISTENER.lsnr           ONLINE ONLINE asmdvl01 STABLE

    ora.asm                     ONLINE ONLINE asmdvl01 Started,STABLE

    ora.ons                     OFFLINE OFFLINE asmdvl01 STABLE

    --------------------------------------------------------------------------

    Cluster Resources

    --------------------------------------------------------------------------

    ora.cssd                    

    1                           ONLINE ONLINE asmdvl01 STABLE

    ora.diskmon                 

    1                           OFFLINE OFFLINE STABLE

    ora.evmd                    

    1                           ONLINE ONLINE asmdvl01 STABLE

    ora.sundev.db  

    1                           ONLINE ONLINE asmdvl01 Open,HOME=/d01/app/o

                                                       racle/product/19.3/d

                                                       b_home,STABLE

    --------------------------------------------------------------------------

  
2) bring down the listener first  

    [root@asmdvl01 ~]# lsnrctl stop listener

    LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 06-FEB-2020 20:50:24

    Copyright (c) 1991, 2019, Oracle. All rights reserved.

    Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=asmdvl01.seo.int)(PORT=1521)))

    The command completed successfully

  
3) As root user , deconfigured the system using roothas per script.  

    [root@asmdvl01 install]# ./roothas.sh -deconfig -force

    Using configuration parameter file: /a01/app/grid/product/19.3/grid_home/crs/install/crsconfig_params

    The log of current session can be found at:

    /a01/app/grid_base/crsdata/asmdvl01/crsconfig/hadeconfig.log

    2020/02/06 20:52:42 CLSRSC-332: CRS resources for listeners are still configured

    2020/02/06 20:56:01 CLSRSC-337: Successfully deconfigured Oracle Restart stack

  
4) now change the hostname  

    [root@asmdvl01 install]# hostnamectl set-hostname asmtsl01.seo.int

  
5) Edit the /etc/hosts file and add changes accordingly.  
6)As root user reconfigured the system using roothas perl script  

    [root@asmtsl01 install]# ./roothas.sh

    Using configuration parameter file: /a01/app/grid/product/19.3/grid_home/crs/install/crsconfig_params

    The log of current session can be found at:

    /a01/app/grid_base/crsdata/asmtsl01/crsconfig/roothas_2020-02-06_08-59-56PM.log

    2020/02/06 21:00:09 CLSRSC-363: User ignored prerequisites during installation

    LOCAL ADD MODE

    Creating OCR keys for user 'grid', privgrp 'oinstall'..

    Operation successful.

    LOCAL ONLY MODE

    Successfully accumulated necessary OCR keys.

    Creating OCR keys for user 'root', privgrp 'root'..

    Operation successful.

    CRS-4664: Node asmtsl01 successfully pinned.

    2020/02/06 21:00:28 CLSRSC-330: Adding Clusterware entries to file 'oracle-ohasd.service'

    asmtsl01 2020/02/06 21:04:43 /a01/app/grid_base/crsdata/asmtsl01/olr/backup_20200206_210443.olr 724960844

    2020/02/06 21:04:44 CLSRSC-327: Successfully configured Oracle Restart for a standalone server

  

    7)now check the status of resources

    [root@asmtsl01 install]# crsctl stat res -t

    --------------------------------------------------------------------------

    Name Target State Server State details

    --------------------------------------------------------------------------

    Local Resources

    --------------------------------------------------------------------------

    ora.ons  OFFLINE OFFLINE asmtsl01 STABLE

    --------------------------------------------------------------------------

    Cluster Resources

    --------------------------------------------------------------------------

    ora.cssd

    1             OFFLINE OFFLINE STABLE

    ora.diskmon

    1             OFFLINE OFFLINE STABLE

    ora.evmd

    1             ONLINE ONLINE asmtsl01 STABLE

    --------------------------------------------------------------------------------

  
8) Make cssd auto start enabled  

    [root@asmtsl01 install]# $ORACLE_HOME/bin/crsctl modify resource "ora.cssd" -init -attr "AUTO_START=1"

  
9) restart HAS  

    [root@asmtsl01 install]# $ORACLE_HOME/bin/crsctl stop has

    CRS-2791: Starting shutdown of Oracle High Availability Services-managed resources on 'asmtsl01'

    CRS-2673: Attempting to stop 'ora.evmd' on 'asmtsl01'

    CRS-2677: Stop of 'ora.evmd' on 'asmtsl01' succeeded

    CRS-2793: Shutdown of Oracle High Availability Services-managed resources on 'asmtsl01' has completed

    CRS-4133: Oracle High Availability Services has been stopped.

    [root@asmtsl01 install]# $ORACLE_HOME/bin/crsctl start has

    CRS-4123: Oracle High Availability Services has been started.

  

    [root@asmtsl01 install]# crsctl stat res -t

    --------------------------------------------------------------------------

    Name Target State Server State details

    --------------------------------------------------------------------------

    Local Resources

    --------------------------------------------------------------------------

    ora.ons    OFFLINE OFFLINE asmtsl01 STABLE

    --------------------------------------------------------------------------

    Cluster Resources

    --------------------------------------------------------------------------

    ora.cssd

    1          ONLINE ONLINE asmtsl01 STABLE

    ora.diskmon

    1          OFFLINE OFFLINE STABLE

    ora.evmd

    1          ONLINE INTERMEDIATE asmtsl01 STABLE

    --------------------------------------------------------------------------

  

    10) add listener

    [root@asmtsl01 install]# srvctl add listener

  

    [root@asmtsl01 install]# crsctl stat res -t

    --------------------------------------------------------------------------

    Name Target State Server State details

    --------------------------------------------------------------------------

    Local Resources

    --------------------------------------------------------------------------

    ora.LISTENER.lsnr

                      OFFLINE OFFLINE asmtsl01 STABLE

    ora.ons

                      OFFLINE OFFLINE asmtsl01 STABLE

    --------------------------------------------------------------------------

    Cluster Resources

    --------------------------------------------------------------------------

    ora.cssd

    1                  ONLINE ONLINE asmtsl01 STABLE

    ora.diskmon

    1                  OFFLINE OFFLINE STABLE

    ora.evmd

    1                  ONLINE ONLINE asmtsl01 STABLE

    --------------------------------------------------------------------------

  
11) Now start the listener  

    [root@asmtsl01 install]# srvctl start listener

    [root@asmtsl01 install]# crsctl stat res -t

    --------------------------------------------------------------------------

    Name Target State Server State details

    --------------------------------------------------------------------------

    Local Resources

    --------------------------------------------------------------------------

    ora.LISTENER.lsnr

                       ONLINE ONLINE asmtsl01 STABLE

    ora.ons

                       OFFLINE OFFLINE asmtsl01 STABLE

    --------------------------------------------------------------------------

    Cluster Resources

    --------------------------------------------------------------------------

    ora.cssd

    1                  ONLINE ONLINE asmtsl01 STABLE

    ora.diskmon

    1                  OFFLINE OFFLINE STABLE

    ora.evmd

    1                  ONLINE ONLINE asmtsl01 STABLE

    --------------------------------------------------------------------------

  
12) create a pfile for asm  

    [grid@asmtsl01 dbs]$ cat init+ASM.ora

    # Oracle init.ora parameter file generated by instance +ASM on 02/06/2020 21:19:51

    __oracle_base='/a01/app/grid_base' # ORACLE_BASE set from environment

    asm_diskgroups='CRS','DATA','FRA'

    asm_diskstring='/dev/oracleasm/disks/*'

    asm_power_limit=2

    instance_type='asm'

  

    [grid@asmtsl01 dbs]$ srvctl add asm

    [grid@asmtsl01 dbs]$ srvctl start asm

    [grid@asmtsl01 dbs]$ crsctl stat res -t

    --------------------------------------------------------------------------

    Name Target State Server State details

    --------------------------------------------------------------------------

    Local Resources

    --------------------------------------------------------------------------

    ora.CRS.dg

                        ONLINE ONLINE asmtsl01 STABLE

    ora.LISTENER.lsnr

                        ONLINE ONLINE asmtsl01 STABLE

    ora.asm

                        ONLINE ONLINE asmtsl01 Started,STABLE

    ora.ons

                        OFFLINE OFFLINE asmtsl01 STABLE

    --------------------------------------------------------------------------

    Cluster Resources

    --------------------------------------------------------------------------

    ora.cssd

    1                    ONLINE ONLINE asmtsl01 STABLE

    ora.diskmon

    1                    OFFLINE OFFLINE STABLE

    ora.evmd

    1                    ONLINE ONLINE asmtsl01 STABLE

    --------------------------------------------------------------------------

  

    [grid@asmtsl01 dbs]$ asmcmd lsdg

    State Type Rebal Sector Logical_Sector Block AU Total_MB Free_MB Req_mir_free_MB Usable_file_MB Offline_disks Voting _files Name

    MOUNTED EXTERN N 512 512 4096 4194304 5116 5016 0 5016 0

    N CRS/

  
mount rest of the disk group.

\[grid@asmtsl01 dbs\]\$ sqlplus / as sysasm

SQL\*Plus: Release 19.0.0.0.0 - Production on Thu Feb 6 21:14:14 2020  
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle. All rights reserved.  
Connected to:  
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production  
Version 19.3.0.0.0  

    SQL> alter diskgroup data mount;

    Diskgroup altered.

    SQL> alter diskgroup fra mount;

    Diskgroup altered.

  
13)Make asm auto start enabled  

    [grid@asmtsl01 dbs]$ crsctl modify resource "ora.asm" -init -attr "AUTO_START=1"

  
14) create spfile in ASM diskgroup  

    15) [grid@asmtsl01 dbs]$ srvctl modify asm -p '+DATA/spfileASM.ora'

    [grid@asmtsl01 dbs]$ srvctl config asm

    ASM home: <CRS home>

    Password file:

    Backup of Password file:

    ASM listener: LISTENER

    Spfile: +DATA/spfileASM.ora

    ASM diskgroup discovery string: ++no-value-at-resource-creation--never-updated-through-ASM++

  
16) restart has  

    [grid@asmtsl01 dbs]$ crsctl stop has

    CRS-2791: Starting shutdown of Oracle High Availability Services-managed resources on 'asmtsl01'

    CRS-2673: Attempting to stop 'ora.DATA.dg' on 'asmtsl01'

    CRS-2673: Attempting to stop 'ora.FRA.dg' on 'asmtsl01'

    CRS-2673: Attempting to stop 'ora.CRS.dg' on 'asmtsl01'

    CRS-2673: Attempting to stop 'ora.LISTENER.lsnr' on 'asmtsl01'

    CRS-2677: Stop of 'ora.LISTENER.lsnr' on 'asmtsl01' succeeded

    CRS-2677: Stop of 'ora.FRA.dg' on 'asmtsl01' succeeded

    CRS-2677: Stop of 'ora.DATA.dg' on 'asmtsl01' succeeded

    CRS-2677: Stop of 'ora.CRS.dg' on 'asmtsl01' succeeded

    CRS-2673: Attempting to stop 'ora.asm' on 'asmtsl01'

    CRS-2677: Stop of 'ora.asm' on 'asmtsl01' succeeded

    CRS-2673: Attempting to stop 'ora.evmd' on 'asmtsl01'

    CRS-2677: Stop of 'ora.evmd' on 'asmtsl01' succeeded

    CRS-2673: Attempting to stop 'ora.cssd' on 'asmtsl01'

    CRS-2677: Stop of 'ora.cssd' on 'asmtsl01' succeeded

    CRS-2793: Shutdown of Oracle High Availability Services-managed resources on 'asmtsl01' has completed

    CRS-4133: Oracle High Availability Services has been stopped.

    [grid@asmtsl01 dbs]$ crsctl start has

    CRS-4123: Oracle High Availability Services has been started.

    17) as a oracle user add database

    [oracle@asmtsl01 ~]$ srvctl add database -d sundev -o /d01/app/oracle/product/19.3/db_home

  
18) create a pfile with below entries  

    [oracle@asmtsl01 dbs]$ cat initsundev.ora

    spfile='+data/sundev/parameterfile/spfile.265.1030560279'

  
19) start the database.  

    srvctl start database -d sundev
