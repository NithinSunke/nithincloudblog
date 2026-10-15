---
title: "Steps to converting non-asm database to asm database"
author: "Sunke Nithin"
date: 2019-06-24
categories: ["RAC"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/24/steps-to-converting-non-asm-database-to-asm-database/
---

# Steps to converting non-asm database to asm database

1)**source environment**  

    [oracle@rac1 ~]$ . oraenv

    ORACLE_SID = [racdv] ? asmrac

    The Oracle base remains unchanged with value /o01/home/oracle

  
database using filesystem to store datafile.

2)**check the size of database**  

    SQL> select db.name,

    d.df_gbytes datafile_gbytes,

    t.tf_gbytes tempfile_gbytes,

    d.df_gbytes + t.tf_gbytes total_gbytes

    from (select sum(bytes)/(1024*1024*1024) df_gbytes from v$datafile) d,

    (select sum(bytes)/(1024*1024*1024) tf_gbytes from v$tempfile) t,

    v$database db;

    NAME DATAFILE_GBYTES TEMPFILE_GBYTES TOTAL_GBYTES

    --------- --------------- --------------- ------------

    ASMRAC 1.55090332 .01953125 1.57043457

  
3)**connect to asmcmd as grid user and verify do we have enough space in asm to store source database**  

    [oracle@rac1 ~]$ sudo su - grid

    [sudo] password for oracle:

    Last login: Mon Jun 24 18:21:07 IST 2019 on pts/0

    [grid@rac1 ~]$ clear

    [grid@rac1 ~]$ . oraenv

    ORACLE_SID = [grid] ? +ASM1

    The Oracle base has been set to /o01/home/grid

    [grid@rac1 ~]$ asmcmd

    ASMCMD> lsdg

    State Type Rebal Sector Block AU Total_MB Free_MB Req_mir_free_MB Usable_file_MB Offline_disks Voting_files Name

    MOUNTED EXTERN N 512 4096 1048576 5087 4691 0 4691 0 Y CRS/

    MOUNTED EXTERN N 512 4096 1048576 5087 3068 0 3068 0 N DATA/

    MOUNTED EXTERN N 512 4096 1048576 5087 4757 0 4757 0 N FRA/

    ASMCMD>

  
4)**get details of c/r/d/f files**  

    SQL> show parameter control_file

    NAME TYPE VALUE

    ------------------------------------ ----------- ------------------------------

    control_file_record_keep_time integer 7

    control_files string /o01/home/oracle/oradata/asmra

    c/control01.ctl, /o01/home/ora

    cle/fra/asmrac/control02.ctl

    SQL> select name from v$datafile;

    NAME

    --------------------------------------------------------------------------------

    /o01/home/oracle/oradata/asmrac/system01.dbf

    /o01/home/oracle/oradata/asmrac/sysaux01.dbf

    /o01/home/oracle/oradata/asmrac/undotbs01.dbf

    /o01/home/oracle/oradata/asmrac/users01.dbf

    /o01/home/oracle/oradata/asmrac/example01.dbf

    SQL> select member from v$logfile;

    MEMBER

    --------------------------------------------------------------------------------

    /o01/home/oracle/oradata/asmrac/redo03.log

    /o01/home/oracle/oradata/asmrac/redo02.log

    /o01/home/oracle/oradata/asmrac/redo01.log

    SQL> select name from v$tempfile;

    NAME

    --------------------------------------------------------------------------------

    /o01/home/oracle/oradata/asmrac/temp01.dbf

  
5)**backup pfile and spfile;**  

    [oracle@rac1 dbs]$ sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Mon Jun 24 19:24:35 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to:

    Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, Real Application Clusters, OLAP, Data Mining

    and Real Application Testing options

    SQL> create pfile from spfile;

    File created.

    SQL> exit

    Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, Real Application Clusters, OLAP, Data Mining

    and Real Application Testing options

    [oracle@rac1 dbs]$ ls -ltr initasmrac.ora

    -rw-r--r-- 1 oracle asmadmin 834 Jun 24 19:24 initasmrac.ora

    [oracle@rac1 dbs]$ cp spfileasmrac.ora initasmrac.ora /home/oracle/

  
6)**make nesseary paramters changes to keep the database in asm**  

    SQL> show parameter control_file

    NAME TYPE VALUE

    ------------------------------------ ----------- ------------------------------

    control_file_record_keep_time integer 7

    control_files string /o01/home/oracle/oradata/asmra

    c/control01.ctl, /o01/home/ora

    cle/fra/asmrac/control02.ctl

    SQL> alter system set control_files='+DATA' scope=spfile;

    System altered.

    SQL> show parameter db_create

    NAME TYPE VALUE

    ------------------------------------ ----------- ------------------------------

    db_create_file_dest string

    db_create_online_log_dest_1 string

    db_create_online_log_dest_2 string

    db_create_online_log_dest_3 string

    db_create_online_log_dest_4 string

    db_create_online_log_dest_5 string

    SQL> alter system set db_create_file_dest='+DATA' scope=spfile;

    System altered.

    SQL> show parameter db_recovery

    NAME TYPE VALUE

    ------------------------------------ ----------- ------------------------------

    db_recovery_file_dest string /o01/home/oracle/fra

    db_recovery_file_dest_size big integer 4182M

    SQL> alter system set db_recovery_file_dest='+FRA' scope=spfile;

    System altered.

  
7)**shutdown and start database in nomount**  

    [oracle@rac1 dbs]$ sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Mon Jun 24 19:29:40 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to:

    Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, Real Application Clusters, OLAP, Data Mining

    and Real Application Testing options

    SQL> shut immediate

    Database closed.

    Database dismounted.

    ORACLE instance shut down.

    SQL> startup nomount

    ORACLE instance started.

    Total System Global Area 1023004672 bytes

    Fixed Size 2259640 bytes

    Variable Size 641729864 bytes

    Database Buffers 373293056 bytes

    Redo Buffers 5722112 bytes

    SQL> exit

    Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, Real Application Clusters, OLAP, Data Mining

    and Real Application Testing options

  
8)**restore controlfile to asm**  

    [oracle@rac1 asmrac]$ rman target /

    Recovery Manager: Release 11.2.0.4.0 - Production on Mon Jun 24 20:01:11 2019 Copyright (c) 1982, 2011, Oracle and/or its affiliates. All rights reserved.

    connected to target database: ASMRAC (not mounted)

    RMAN> restore controlfile from '/o01/home/oracle/oradata/asmrac/control01.ctl';

    Starting restore at 24-JUN-19

    using target database control file instead of recovery catalog

    allocated channel: ORA_DISK_1

    channel ORA_DISK_1: SID=30 device type=DISK

    channel ORA_DISK_1: copied control file copy

    output file name=+DATA/asmrac/controlfile/current.269.1011814247

    Finished restore at 24-JUN-19

    9) mount the database

    SQL> alter database mount;

    Database altered.

    10)backup database as copy to asm

    RMAN> backup as copy database format '+DATA';

    Starting backup at 24-JUN-19

    Starting implicit crosscheck backup at 24-JUN-19

    using target database control file instead of recovery catalog

    allocated channel: ORA_DISK_1

    channel ORA_DISK_1: SID=1 device type=DISK

    Finished implicit crosscheck backup at 24-JUN-19

    Starting implicit crosscheck copy at 24-JUN-19

    using channel ORA_DISK_1

    Finished implicit crosscheck copy at 24-JUN-19

    searching for all files in the recovery area

    cataloging files...

    no files cataloged

    using channel ORA_DISK_1

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00001 name=/o01/home/oracle/oradata/asmrac/system01.dbf

    output file name=+DATA/asmrac/datafile/system.270.1011814335 tag=TAG20190624T193214 RECID=2 STAMP=1011814440

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:01:55

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00002 name=/o01/home/oracle/oradata/asmrac/sysaux01.dbf

    output file name=+DATA/asmrac/datafile/sysaux.271.1011814451 tag=TAG20190624T193214 RECID=3 STAMP=1011814486

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:45

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00005 name=/o01/home/oracle/oradata/asmrac/example01.dbf

    output file name=+DATA/asmrac/datafile/example.272.1011814497 tag=TAG20190624T193214 RECID=4 STAMP=1011814521

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:35

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00003 name=/o01/home/oracle/oradata/asmrac/undotbs01.dbf

    output file name=+DATA/asmrac/datafile/undotbs1.273.1011814531 tag=TAG20190624T193214 RECID=5 STAMP=1011814539

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:15

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00004 name=/o01/home/oracle/oradata/asmrac/users01.dbf

    output file name=+DATA/asmrac/datafile/users.274.1011814547 tag=TAG20190624T193214 RECID=6 STAMP=1011814547

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:01

    Finished backup at 24-JUN-19

  
11)**switch database to copy**  

    RMAN> switch database to copy;

    datafile 1 switched to datafile copy "+DATA/asmrac/datafile/system.270.1011814335"

    datafile 2 switched to datafile copy "+DATA/asmrac/datafile/sysaux.271.1011814451"

    datafile 3 switched to datafile copy "+DATA/asmrac/datafile/undotbs1.273.1011814531"

    datafile 4 switched to datafile copy "+DATA/asmrac/datafile/users.274.1011814547"

    datafile 5 switched to datafile copy "+DATA/asmrac/datafile/example.272.1011814497"

    RMAN> run {

    set newname for tempfile 1 to '+DATA';

    switch tempfile all;

    }2> 3> 4>

    executing command: SET NEWNAME

    renamed tempfile 1 to +DATA in control file

    RMAN>

    RMAN> exit

  
12) **create new redolog in each group to asm and drop filesystem redolog**  

    SQL> select member from v$logfile;

    MEMBER

    --------------------------------------------------------------------------------

    /o01/home/oracle/oradata/asmrac/redo03.log

    /o01/home/oracle/oradata/asmrac/redo02.log

    /o01/home/oracle/oradata/asmrac/redo01.log

    SQL> alter database add logfile member '+DATA' to group 1;

    Database altered.

    SQL> alter database add logfile member '+DATA' to group 2;

    Database altered.

    SQL> alter database add logfile member '+DATA' to group 3;

    Database altered.

    SQL> select member from v$logfile;

    MEMBER

    --------------------------------------------------------------------------------

    /o01/home/oracle/oradata/asmrac/redo03.log

    /o01/home/oracle/oradata/asmrac/redo02.log

    /o01/home/oracle/oradata/asmrac/redo01.log

    +DATA/asmrac/onlinelog/group_1.279.1011816477

    +DATA/asmrac/onlinelog/group_2.280.1011816489

    +DATA/asmrac/onlinelog/group_3.281.1011816497

    6 rows selected.

  
**now drop the filesystem redo logfiles**  

    SQL> alter database drop logfile member '/o01/home/oracle/oradata/asmrac/redo01.log';

    alter database drop logfile member '/o01/home/oracle/oradata/asmrac/redo01.log'

    *

    ERROR at line 1:

    ORA-00362: member is required to form a valid logfile in group 1

    ORA-01517: log member: '/o01/home/oracle/oradata/asmrac/redo01.log'

    SQL> alter system switch logfile;

    System altered.

    SQL> /

    System altered.

    SQL> /

    System altered.

    SQL> alter database drop logfile member '/o01/home/oracle/oradata/asmrac/redo01.log';

    Database altered.

    SQL> alter database drop logfile member '/o01/home/oracle/oradata/asmrac/redo02.log';

    Database altered.

    SQL> alter database drop logfile member '/o01/home/oracle/oradata/asmrac/redo03.log';

    alter database drop logfile member '/o01/home/oracle/oradata/asmrac/redo03.log'

    *

    ERROR at line 1:

    ORA-01609: log 3 is the current log for thread 1 - cannot drop members

    ORA-00312: online log 3 thread 1: '/o01/home/oracle/oradata/asmrac/redo03.log'

    ORA-00312: online log 3 thread 1:

    '+DATA/asmrac/onlinelog/group_3.281.1011816497'

    SQL> alter system switch logfile;

    System altered.

    SQL> alter database drop logfile member '/o01/home/oracle/oradata/asmrac/redo03.log';

    Database altered.

    SQL> select member from v$logfile;

    MEMBER

    --------------------------------------------------------------------------------

    +DATA/asmrac/onlinelog/group_1.279.1011816477

    +DATA/asmrac/onlinelog/group_2.280.1011816489

    +DATA/asmrac/onlinelog/group_3.281.1011816497

    SQL>

  
14) **Finally check the FRA and change it to ASM location if not aleady done.**  

    SQL> show parameter db_recovery

    NAME TYPE VALUE

    ------------------------------------ ----------- 

    db_recovery_file_dest string           +FRA

    db_recovery_file_dest_size big integer 4182M

  
15) **verify the environment**  

    SQL> show parameter control

    NAME TYPE VALUE

    ------------------------------------ ----------- ------------------------------

    control_file_record_keep_time integer 7

    control_files string +DATA/asmrac/controlfile/curre

    nt.269.1011814247

    control_management_pack_access string DIAGNOSTIC+TUNING

    SQL> select name from v$datafile;

    NAME

    --------------------------------------------------------------------------------

    +DATA/asmrac/datafile/system.270.1011814335

    +DATA/asmrac/datafile/sysaux.271.1011814451

    +DATA/asmrac/datafile/undotbs1.273.1011814531

    +DATA/asmrac/datafile/users.274.1011814547

    +DATA/asmrac/datafile/example.272.1011814497

    SQL> select name from v$tempfile;

    NAME

    --------------------------------------------------------------------------------

    +DATA/asmrac/tempfile/temp.278.1011816197

    SQL> select member from v$logfile;

    MEMBER

    --------------------------------------------------------------------------------

    +DATA/asmrac/onlinelog/group_1.279.1011816477

    +DATA/asmrac/onlinelog/group_2.280.1011816489

    +DATA/asmrac/onlinelog/group_3.281.1011816497
