---
title: "Add New ASM Disk To Diskgroup"
author: "Sunke Nithin"
date: 2019-06-26
categories: ["RAC"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/26/add-new-asm-disk-to-diskgroup/
---

# Add New ASM Disk To Diskgroup

High keve steps to add disk to diskgroup  

1.  Collect the details from the environment
2.  Verify the LUN details
3.  Create asm disk
4.  Verify ASM disk is properly created and ready to add to disk group
5.  Add disk to disk group
6.  Verify the re-balancing operation
7.  Verify the details from both the ASM instances

  
**1) Collect the details from the environment**  

    [grid@orrcdbdvl01 ~]$ crs_stat

    SQL> select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    SQL> select group_number,name,state,type from v$asm_diskgroup;

    SQL> select group_number, name, TOTAL_MB, FREE_MB from V$asm_disk_stat;

    SQL> select * from v$asm_operation;

  
 

**2) Verify the LUN details**  

    [root@dbdvl01 ~]# /etc/init.d/oracleasm listdisks

    [root@dbdvl01 ~]# fdisk -l /dev/mapper/vpath3

    [root@dbdvl01 ~]# multipath -ll

    [root@dbdvl01 ~]# ll /dev/mpath/

    [root@dbdvl01 ~]# ll /dev/mapper/

  
**3) Create asm disk**

**On Node01:**  

    [root@dbdvl01 ~]# /etc/init.d/oracleasm createdisk DATA2 /dev/mapper/vpath3

    [root@dbdvl01 ~]# /etc/init.d/oracleasm listdisks

  
**On Node02:**  

    [root@dbdvl02 ~]# /etc/init.d/oracleasm scandisks

    [root@dbdvl02 ~]# /etc/init.d/oracleasm listdisks

  
**4) Verify ASM disk is properly created and ready to add to disk group**  

    [grid@dbdvl01 ~]$ sqlplus '/as sysdba'

    SQL> select path from v$asm_disk;

    SQL> select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    SQL> select MOUNT_STATUS,HEADER_STATUS,MODE_STATUS,state from v$asm_disk;

  
NOTE: It should show HEADER_STATUS for asm disk as PROVISIONED  

    SQL> select disk_number,failgroup,create_date,path,name from v$asm_disk;

    SQL> select group_number,name,state,type from v$asm_diskgroup;

  
**5) Add disk to disk group**  

    SQL> conn /as sysasm

    SQL> select group_number,name,state,type from v$asm_diskgroup;

    SQL> alter diskgroup DEVSHR_DATA add disk 'ORCL:DATA2';

  
**6) Verify the re-balancing operation**  

    SQL> select * from v$asm_operation;

  
**7) Verify the details from both the ASM instances**  

    SQL> select disk_number,failgroup,create_date,path,name from v$asm_disk;

    SQL> select MOUNT_STATUS,HEADER_STATUS,MODE_STATUS,state from v$asm_disk;

    SQL> select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    SQL> select group_number, name, TOTAL_MB, FREE_MB from V$asm_disk_stat;

  
**SessionLog For Step1:**  

    [grid@dbdvl01 ~]$crs_stat

    HA Resource                             Target     State

    -----------                             ------     -----

    ora.DEVSHR_DATA.dg                      ONLINE     ONLINE on dbdvl01

    ora.FLASH.dg                            ONLINE     ONLINE on dbdvl01

    ora.LISTENER.lsnr                       ONLINE     ONLINE on dbdvl01

    ora.LISTENER_SCAN1.lsnr                 ONLINE     ONLINE on dbdvl02

    ora.LISTENER_SCAN2.lsnr                 ONLINE     ONLINE on dbdvl01

    ora.LISTENER_SCAN3.lsnr                 ONLINE     ONLINE on dbdvl01

    ora.OCR_VOTE.dg                         ONLINE     ONLINE on dbdvl01

    ora.asm                                 ONLINE     ONLINE on dbdvl01

    ora.cvu                                 ONLINE     ONLINE on dbdvl02

    ora.gsd                                 OFFLINE    OFFLINE

    ora.net1.network                        ONLINE     ONLINE on dbdvl01

    ora.oc4j                                ONLINE     ONLINE on dbdvl01

    ora.ons                                 ONLINE     ONLINE on dbdvl01

    ora.dbdvl01.ASM1.asm                    ONLINE     ONLINE on dbdvl01

    ora.dbdvl01.LISTENER_DBDVL01.lsnr       ONLINE     ONLINE on dbdvl01

    ora.dbdvl01.gsd                         OFFLINE    OFFLINE

    ora.dbdvl01.ons                         ONLINE     ONLINE on dbdvl01

    ora.dbdvl01.vip                         ONLINE     ONLINE on dbdvl01

    ora.dbdvl02.ASM2.asm                    ONLINE     ONLINE on dbdvl02

    ora.dvl02.LISTENER_DBDVL02.lsnr         ONLINE     ONLINE on dbdvl02

    ora.dbdvl02.gsd                         OFFLINE    OFFLINE

    ora.dbdvl02.ons                         ONLINE     ONLINE on dbdvl02

    ora.dbdvl02.vip                         ONLINE     ONLINE on dbdvl02

    ora.registry.acfs                       ONLINE     ONLINE on dbdvl01

    ora.scan1.vip                           ONLINE     ONLINE on dbdvl02

    ora.scan2.vip                           ONLINE     ONLINE on dbdvl01

    ora.scan3.vip                           ONLINE     ONLINE on dbdvl01

    ora.shrcdv.db                           ONLINE     ONLINE on dbdvl01

    ora.shrcdv.shrcdv_service.svc           ONLINE     ONLINE on dbdvl01

  
\[grid@dbdvl01 ~\]\$ sqlplus '/as sysdba'

SQL\*Plus: Release 11.2.0.2.0 Production on Thu Sep 12 02:04:40 2013

Copyright (c) 1982, 2010, Oracle.  All rights reserved.

Connected to:

Oracle Database 11g Enterprise Edition Release 11.2.0.2.0 - 64bit Production

With the Real Application Clusters and Automatic Storage Management options

   

     col GROUP_NUMBER for 9999

     col NAME for a15

     col path for a25

     col state for a10

     set linesize 100

     select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    GROUP_NUMBER NAME            PATH                      STATE

    ------------ --------------- ------------------------- ----------

    3            CRSVOL1         ORCL:CRSVOL1              NORMAL

    1            DATA1           ORCL:DATA1                NORMAL

    2            FRADISK1        ORCL:FRADISK1             NORMAL

  

    SQL> select group_number,name,state,type from v$asm_diskgroup;

    GROUP_NUMBER NAME            STATE      TYPE

    ------------ --------------- ---------- ------

    1             DEVSHR_DATA     MOUNTED    EXTERN

    2             FLASH           MOUNTED    EXTERN

    3             OCR_VOTE        MOUNTED    EXTERN

  
   

    SQL> select group_number, name, TOTAL_MB, FREE_MB from $asm_disk_stat;

    GROUP_NUMBER NAME              TOTAL_MB    FREE_MB

    ------------ --------------- ---------- ----------

    3             CRSVOL1              10240       9844

    1             DATA1               102400      14680

    2             FRADISK1            102400     101968

  
   

    SQL> select * from v$asm_operation;

    no rows selected

  
**Sessionlog For Step 2:**  

    [root@dbdvl01 ~]# /etc/init.d/oracleasm listdisks

    CRSVOL1

    DATA1

    FRADISK1

  

    [root@dbdvl01 ~]# fdisk -l /dev/mapper/vpath3

    Disk /dev/mapper/vpath3: 214.7 GB, 214748364800 bytes

    255 heads, 63 sectors/track, 26108 cylinders

    Units = cylinders of 16065 * 512 = 8225280 bytes

    Disk /dev/mapper/vpath3 doesn't contain a valid partition table

  

    [root@dbdvl01 ~]# multipath -ll

    vpath3 (360050768019600fba800000000000572) dm-13 IBM,2145

    [size=200G][features=1 queue_if_no_path][hwhandler=0][rw]

    _ round-robin 0 [prio=200][active]

    _ 1:0:0:3 sdaa 65:160 [active][ready]

    _ 1:0:1:3 sdab 65:176 [active][ready]

    _ 2:0:0:3 sdae 65:224 [active][ready]

    _ 2:0:1:3 sdaf 65:240 [active][ready]

    _ round-robin 0 [prio=40][enabled]

    _ 1:0:2:3 sdac 65:192 [active][ready]

    _ 1:0:3:3 sdad 65:208 [active][ready]

    _ 2:0:2:3 sdag 66:0   [active][ready]

    _ 2:0:3:3 sdah 66:16  [active][ready]

    vpath2 (360050768019600fba8000000000004ea) dm-12 IBM,2145

    [size=100G][features=1 queue_if_no_path][hwhandler=0][rw]

    _ round-robin 0 [prio=200][active]

    _ 1:0:2:0 sdh  8:112  [active][ready]

    _ 1:0:3:0 sdk  8:160  [active][ready]

    _ 2:0:2:0 sdt  65:48  [active][ready]

    _ 2:0:3:0 sdw  65:96  [active][ready]

    _ round-robin 0 [prio=40][enabled]

    _ 1:0:0:0 sdb  8:16   [active][ready]

    _ 1:0:1:0 sde  8:64   [active][ready]

    _ 2:0:0:0 sdn  8:208  [active][ready]

    _ 2:0:1:0 sdq  65:0   [active][ready]

    vpath1 (360050768019600fba8000000000004eb) dm-10 IBM,2145

    [size=100G][features=1 queue_if_no_path][hwhandler=0][rw]

    _ round-robin 0 [prio=200][active]

    _ 1:0:0:1 sdc  8:32   [active][ready]

    _ 1:0:1:1 sdf  8:80   [active][ready]

    _ 2:0:0:1 sdo  8:224  [active][ready]

    _ 2:0:1:1 sdr  65:16  [active][ready]

    _ round-robin 0 [prio=40][enabled]

    _ 1:0:2:1 sdi  8:128  [active][ready]

    _ 1:0:3:1 sdl  8:176  [active][ready]

    _ 2:0:2:1 sdu  65:64  [active][ready]

    _ 2:0:3:1 sdx  65:112 [active][ready]

    vpath0 (360050768019600fba8000000000004ec) dm-11 IBM,2145

    [size=10G][features=1 queue_if_no_path][hwhandler=0][rw]

    _ round-robin 0 [prio=200][active]

    _ 1:0:2:2 sdj  8:144  [active][ready]

    _ 1:0:3:2 sdm  8:192  [active][ready]

    _ 2:0:2:2 sdv  65:80  [active][ready]

    _ 2:0:3:2 sdy  65:128 [active][ready]

    _ round-robin 0 [prio=40][enabled]

    _ 1:0:0:2 sdd  8:48   [active][ready]

    _ 1:0:1:2 sdg  8:96   [active][ready]

    _ 2:0:0:2 sdp  8:240  [active][ready]

    _ 2:0:1:2 sds  65:32  [active][ready]

  

    [root@dbdvl01 ~]# ll /dev/mpath/

    total 0

    lrwxrwxrwx 1 root root 8 Sep 11 11:40 360050768019600fba800000000000572 -> ../dm-13

    lrwxrwxrwx 1 root root 8 Jul 20  2012 vpath0 -> ../dm-11

    lrwxrwxrwx 1 root root 8 Jul 20  2012 vpath1 -> ../dm-10

    lrwxrwxrwx 1 root root 8 Jul 20  2012 vpath2 -> ../dm-12

  

    [root@dbdvl01 ~]# ll /dev/mapper/

    total 0

    crw------- 1 root root  10, 63 Jul 20  2012 control

    brw-rw---- 1 root disk 253,  6 Jul 20  2012 vg00-bmclv

    brw-rw---- 1 root disk 253,  1 Jul 20  2012 vg00-homelv

    brw-rw---- 1 root disk 253,  8 Jul 20  2012 vg00-o001lv

    brw-rw---- 1 root disk 253,  9 Jul 20  2012 vg00-oemlv

    brw-rw---- 1 root disk 253,  2 Jul 20  2012 vg00-optlv

    brw-rw---- 1 root disk 253,  0 Jul 20  2012 vg00-rootlv

    brw-rw---- 1 root disk 253,  7 Jul 20  2012 vg00-swap01lv

    brw-rw---- 1 root disk 253,  5 Jul 20  2012 vg00-tmplv

    brw-rw---- 1 root disk 253,  3 Jul 20  2012 vg00-usrlv

    brw-rw---- 1 root disk 253,  4 Jul 20  2012 vg00-varlv

    brw-rw---- 1 root disk 253, 11 Jul 20  2012 vpath0

    brw-rw---- 1 root disk 253, 10 Jul 20  2012 vpath1

    brw-rw---- 1 root disk 253, 12 Jul 20  2012 vpath2

    brw-rw---- 1 root disk 253, 13 Sep 11 11:40 vpath3

  
**SessionLog For Step 3:**

**On Node01:**  

    [root@dbdvl01 ~]# /etc/init.d/oracleasm createdisk DATA2 /dev/mapper/vpath3

    Marking disk "DATA2" as an ASM disk:                       [  OK  ]

    [root@dbdvl01 ~]# /etc/init.d/oracleasm scandisks

    Scanning the system for Oracle ASMLib disks:               [  OK  ]

  

    [root@dbdvl01 ~]# /etc/init.d/oracleasm listdisks

    CRSVOL1

    DATA1

    DATA2

    FRADISK1

  
**On Node02:**  

    [root@dbdvl02 ~]# /etc/init.d/oracleasm listdisks

    CRSVOL1

    DATA1

    FRADISK1

  

    [root@dbdvl02 ~]# /etc/init.d/oracleasm scandisks

    Scanning the system for Oracle ASMLib disks:               [  OK  ]

    [root@dbdvl02 ~]# /etc/init.d/oracleasm listdisks

    CRSVOL1

    DATA1

    DATA2

    FRADISK1

  
**SessionLog For Step 4:**

\[grid@dbdvl01 ~\]\$ sqlplus '/as sysdba'

SQL\*Plus: Release 11.2.0.2.0 Production on Thu Sep 12 02:25:54 2013

Copyright (c) 1982, 2010, Oracle.  All rights reserved.

Connected to:

Oracle Database 11g Enterprise Edition Release 11.2.0.2.0 - 64bit Production

With the Real Application Clusters and Automatic Storage Management options

   

    SQL> select path from v$asm_disk;

    PATH

    -----------------------------------------------------------------

    ORCL:DATA2

    ORCL:CRSVOL1

    ORCL:DATA1

    ORCL:FRADISK1

  
   

    SQL> col GROUP_NUMBER for 9999

    SQL> col NAME for a15

    SQL> col path for a25

    SQL> col state for a10

    SQL> set linesize 100

    SQL> select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    GROUP_NUMBER NAME            PATH                      STATE

    ------------ --------------- ------------------------- ----------

    0                 ORCL:DATA2                NORMAL

    3 CRSVOL1         ORCL:CRSVOL1              NORMAL

    1 DATA1           ORCL:DATA1                NORMAL

    2 FRADISK1        ORCL:FRADISK1             NORMAL

  
   

    SQL> select MOUNT_STATUS,HEADER_STATUS,MODE_STATUS,state from v$asm_disk;

    MOUNT_S HEADER_STATU MODE_ST STATE

    ------- ------------ ------- ----------

    CLOSED  PROVISIONED  ONLINE  NORMAL

    CACHED  MEMBER       ONLINE  NORMAL

    CACHED  MEMBER       ONLINE  NORMAL

    CACHED  MEMBER       ONLINE  NORMAL

  
   

    SQL> select disk_number,failgroup,create_date,path,name from v$asm_disk;

    DISK_NUMBER FAILGROUP   CREATE_DATE PATH                      NAME

    ----------- ----------- ----------- ------------------- ---------------

    0                                    ORCL:DATA2

    0             CRSVOL1    17-JUL-12   ORCL:CRSVOL1              CRSVOL1

    0             DATA1      17-JUL-12   ORCL:DATA1                DATA1

    0             FRADISK1   17-JUL-12   ORCL:FRADISK1             FRADISK1

  

    SQL> select group_number,name,state,type from v$asm_diskgroup;

    GROUP_NUMBER NAME            STATE      TYPE

    ------------ --------------- ---------- ------

    1            DEVSHR_DATA     MOUNTED    EXTERN

    2            FLASH           MOUNTED    EXTERN

    3            OCR_VOTE        MOUNTED    EXTERN

  
**SessionLog For Step 5:**

SQL\> conn /as sysasm

Connected.  

    SQL> select group_number,name,state,type from v$asm_diskgroup;

    GROUP_NUMBER NAME            STATE      TYPE

    ------------ --------------- ---------- ------

    1            DEVSHR_DATA     MOUNTED    EXTERN

    2            FLASH           MOUNTED    EXTERN

    3            OCR_VOTE        MOUNTED    EXTERN

  
   

    SQL> alter diskgroup DEVSHR_DATA add disk 'ORCL:DATA2';

    Diskgroup altered.

  
**SessionLog For Step 6:**  

    SQL> select * from v$asm_operation;

    GROUP_NUMBER OPERA STATE POWER ACTUAL SOFAR EST_WORK EST_RATE EST_MINUTES  

    ------------------------------------------------------------------------

    1            REBAL RUN    1       1     0   58479          0           0

  
**Sessionlog For Step 7:**  

    SQL> select disk_number,failgroup,create_date,path,name from v$asm_disk;

    DISK_NUMBER FAILGROUP CREATE_DATE PATH             NAME

    ----------- ------------------------------------------------- 

    0            CRSVOL1  17-JUL-12   ORCL:CRSVOL1      CRSVOL1

    0            DATA1    17-JUL-12   ORCL:DATA1        DATA1

    0            FRADISK1 17-JUL-12   ORCL:FRADISK1     FRADISK1

    1            DATA2    12-SEP-13   ORCL:DATA2        DATA2

  

    SQL> select MOUNT_STATUS,HEADER_STATUS,MODE_STATUS,state from v$asm_disk;

    MOUNT_S HEADER_STATU MODE_ST STATE

    ------- ------------ ------- ----------

    CACHED  MEMBER       ONLINE  NORMAL

    CACHED  MEMBER       ONLINE  NORMAL

    CACHED  MEMBER       ONLINE  NORMAL

    CACHED  MEMBER       ONLINE  NORMAL

  

    SQL> select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    GROUP_NUMBER NAME            PATH                      STATE

    ------------ --------------- ------------------------- ----------

    3            CRSVOL1         ORCL:CRSVOL1              NORMAL

    1            DATA1           ORCL:DATA1                NORMAL

    2            FRADISK1        ORCL:FRADISK1             NORMAL

    1            DATA2           ORCL:DATA2                NORMAL

  

    SQL> select group_number, name, TOTAL_MB, FREE_MB from V$asm_disk_stat;

    GROUP_NUMBER NAME              TOTAL_MB    FREE_MB

    ------------ --------------- ---------- ----------

    3            CRSVOL1              10240       9844

    1            DATA1               102400      49229

    2            FRADISK1            102400     101967

    1            DATA2               204800     170248
