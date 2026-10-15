---
title: "Moving ocr&voting disk  from the present lun to the new lun."
author: "Sunke Nithin"
date: 2019-06-26
categories: ["RAC"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/26/lun-migration-2/
---

# Moving ocr&voting disk  from the present lun to the new lun.

Moving ocr&voting disk  from the present lun to the new lun.

    Action Plan To new asm disk and create a new asm diskgroup

     Collect the details from the environment

1.  Verify the LUN details

2.  Create asm disk

3.  Verify ASM disk is properly created and ready to add to disk group

4.  Add disk to disk group

5.  Verify the re-balancing operation

6.  Verify the details from both the ASM instances

&nbsp;

    1) Collect the details from the environment

    [grid@dbdvl01 ~]$ Check_CRS.txt

    SQL> select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    SQL> select group_number,name,state,type from v$asm_diskgroup;

    SQL> select group_number, name, TOTAL_MB, FREE_MB from V$asm_disk_stat;

    SQL> select * from v$asm_operation;

    2) Verify the LUN details

    [root@dbdvl01 ~]# /etc/init.d/oracleasm listdisks

    [root@dbdvl01 ~]# fdisk -l /dev/mapper/vpath3

    3) Create asm disk

    On Node01:

    [root@dbdvl01 ~]# /etc/init.d/oracleasm createdisk DATA2 /dev/mapper/vpath3

    [root@dbdvl01 ~]# /etc/init.d/oracleasm listdisks

    On Node02:

    [root@dbdvl02 ~]# /etc/init.d/oracleasm listdisks
    [root@dbdvl02 ~]# /etc/init.d/oracleasm scandisks
    [root@dbdvl02 ~]# /etc/init.d/oracleasm listdisks

    4) Verify ASM disk is properly created and ready to add to disk group

    [grid@dbdvl01 ~]$ sqlplus '/as sysdba'

    SQL> select path from v$asm_disk;

    SQL> select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    SQL> select MOUNT_STATUS,HEADER_STATUS,MODE_STATUS,state from v$asm_disk;

    NOTE: It should show HEADER_STATUS for asm disk as PROVISIONED

    SQL> select disk_number,failgroup,create_date,path,name from v$asm_disk;

    SQL> select group_number,name,state,type from v$asm_diskgroup;

    5) Add disk to disk group

    SQL> conn /as sysasm

    SQL> select group_number,name,state,type from v$asm_diskgroup;

    SQL> alter diskgroup NEW_OCRVD add disk 'ORCL:OCRVD2';

    6) Verify the re-balancing operation

    SQL> select * from v$asm_operation;

    7) Verify the details from both the ASM instances

    SQL> select disk_number,failgroup,create_date,path,name from v$asm_disk;

    SQL> select MOUNT_STATUS,HEADER_STATUS,MODE_STATUS,state from v$asm_disk;

    SQL> select GROUP_NUMBER,NAME,PATH,STATE from v$asm_disk;

    SQL> select group_number, name, TOTAL_MB, FREE_MB from V$asm_disk_stat;

    Now steps to move ocr and voting disk to new asm diskgroups(New luns).

    STEP1:

    Create a new diskgroup for OCR and Voting disk

    Use ascma and create new Data group named OCR and verify that this datagroup is mounted

    $ asmcmd lsdg

    State    Type    Rebal  Sector  Block       AU  Total_MB  Free_MB  Req_mir_free_MB  Usable_file_MB  Offline_disks  Voting_files  NameMOUNTED  NORMAL  N         512   4096  1048576     40944    25698            10236            7731              0             Y  DATA/MOUNTED  NORMAL  N         512   4096  1048576      6141     5730             2047            1841              0             N  OCR

    Attention:  To avoid error CRS-4602: Failed 27 to add voting file .. during running $GRID_HOME/bin/crsctl  replace votedisk  double check that the  the newly created diskgroup is mounted on any cluster instances by running

    $ asmcmd lsdg
    $ asmcmd lsdsk

    on each instance.

    STEP2:

    Relocate OCR from +DATA to +OCR diskgroup

    Check the current OCR location

    # $GRID_HOME/bin/ocrcheck

    Status of Oracle Cluster Registry is as follows :

    Version                  :          3

    Total space (kbytes)     :     262120

    Used space (kbytes)      :       3664

    Available space (kbytes) :     258456

    ID                       :  630679368

    Device/File Name         :      +DATA

                                        Device/File integrity check succeeded

    Add +OCR datagroup as new OCR location and verify status

    #  $GRID_HOME/bin/ocrconfig -add +OCR

    #  $GRID_HOME/bin/ocrcheck

    Status of Oracle Cluster Registry is as follows :

    Version                  :          3

    Total space (kbytes)     :     262120

    Used space (kbytes)      :       3664

    Available space (kbytes) :     258456

    ID                       :  630679368

    Device/File Name         :      +DATA

                                        Device/File integrity check succeeded

         Device/File Name         :       +OCR

                                        Device/File integrity check succeeded

    Delete the old OCR location 

    # $GRID_HOME/bin/ocrconfig -delete +DATA

    # $GRID_HOME/bin/ocrcheck

    Status of Oracle Cluster Registry is as follows :

    Version                  :          3

    Total space (kbytes)     :     262120

    Used space (kbytes)      :       3664

    Available space (kbytes) :     258456

    ID                       :  630679368

     Device/File Name         :       +OCR

                                        Device/File integrity check succeeded

    Relocate voting disks to a new ASM diskgroup

    Check current status :

    # $GRID_HOME/bin/crsctl   query css votedisk

    STATE File Universal Id File Name Disk group
    -- ----- ----------------- --------- ---------

    1. ONLINE   7c7a1eb193644fc3bffd655ae8e43462 (/dev/asmdisk1_udev_sdb1) [DATA] 2. ONLINE   6590ede5237e4f37bfe6aa3d196fe972 (/dev/asmdisk2_udev_sdc1) [DATA] 3. ONLINE   3a7c7b334faf4fdcbf05a246eddf5404 (/dev/asmdisk3_udev_sdd1) [DATA]Located 3 voting disk(s).

    Relocate Voting Disks$  

    $GRID_HOME/bin/crsctl  replace votedisk +OCR

    Successful addition of voting disk 0850980df1024fcfbfc077b9c6935a17.Successful addition of voting disk 0ca6719843734f16bfb62f22f94f399c.Successful addition of voting disk 4383a5ad43324fa2bff759ce20ba0c3f.Successful deletion of voting disk 10c81d1ce5a14fb6bf35cbb22fff3ebf.:1Successful deletion of voting disk 98010612be6b4fc9bf3bc1b186d8758d.Successful deletion of voting disk 9688bec3914d4f70bfc959664ddd8584.

    Successfully replaced voting disk group with +OCR.

    CRS-4266: Voting file(s) successfully replaced

    #  $GRID_HOME/bin/crsctl   query css votedisk
    STATE File Universal Id File Name Disk group
    -- ----- ----------------- --------- ---------
    1. ONLINE b0e94e5d83054fe9bf58b6b98bfacd65 (/dev/asmdisk5_udev_sdf1) [OCR]
    2. ONLINE 88c2a08b4c8c4f85bf0109e0990388e4 (/dev/asmdisk6_udev_sdg1) [OCR]
    3. ONLINE 1108f9a41e814fb2bfed879ff0039dd0 (/dev/asmdisk7_udev_sdh1) [OCR]
