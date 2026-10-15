---
title: "Recover Standby Database using Increment backups."
author: "Sunke Nithin"
date: 2019-06-20
categories: ["RMAN"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/20/recover-standby-database-using-increment-backups/
---

# Recover Standby Database using Increment backups.

Steps:  
==========

primary database side archive generated  

    SQL> SQL> archive log list

    Database log mode Archive Mode

    Automatic archival Enabled

    Archive destination USE_DB_RECOVERY_FILE_DEST

    Oldest online log sequence 40

    Next log sequence to archive 42

    Current log sequence 42

  
standby database side archive applied  

    THREAD# Last Sequence Received Last Sequence Applied Difference

    ---------- ---------------------- --------------------- ----------

    1                     35               35                    1

  
we have log gap of 4 archive log

1)get current scn on primary and standby database  

    primary

    SQL> select current_scn from v$database;

    CURRENT_SCN

    -----------

    1114514

  

    standby

    SQL> select current_scn from v$database;

    CURRENT_SCN

    -----------

    1114207

  
2) stop mrp process on standby database  

    SQL> alter database recover managed standby database cancel;

  
3) shutdown the standby database  

    SQL> shut immediate

    ORA-01109: database not open

    Database dismounted.

    ORACLE instance shut down.

  
4) on primary database take incremental backup from scn  

    RMAN> run

    2> {

    3> allocate channel c1 type disk format '/c01/standby/%U.bkp';

    4> backup incremental from scn 1114207 database;

    5> }

    using target database control file instead of recovery catalog

    allocated channel: c1

    channel c1: SID=41 device type=DISK

    Starting backup at 20-JUN-19

    channel c1: starting full datafile backup set

    channel c1: specifying datafile(s) in backup set

    input datafile file number=00001 name=/b01/oradata/sunkdev/system01.dbf

    input datafile file number=00002 name=/b01/oradata/sunkdev/sysaux01.dbf

    input datafile file number=00005 name=/b01/oradata/sunkdev/example01.dbf

    input datafile file number=00003 name=/b01/oradata/sunkdev/undotbs01.dbf

    input datafile file number=00006 name=/b01/oradata/sunkdev/nithints.dbf

    input datafile file number=00004 name=/b01/oradata/sunkdev/users01.dbf

    channel c1: starting piece 1 at 20-JUN-19

    channel c1: finished piece 1 at 20-JUN-19

    piece handle=/c01/standby/13u4jbu3_1_1.bkp tag=TAG20190620T174123 comment=NONE

    channel c1: backup set complete, elapsed time: 00:00:25

    channel c1: starting full datafile backup set

    channel c1: specifying datafile(s) in backup set

    including current control file in backup set

    channel c1: starting piece 1 at 20-JUN-19

    channel c1: finished piece 1 at 20-JUN-19

    piece handle=/c01/standby/14u4jbut_1_1.bkp tag=TAG20190620T174123 comment=NONE

    channel c1: backup set complete, elapsed time: 00:00:01

    Finished backup at 20-JUN-19

    released channel: c1

    RMAN> exit

  
5) take control file backup from standby database  

    [oracle@oradbdvl01 standby]$ rman target /

    Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jun 20 17:43:00 2019 Copyright (c) 1982, 2011, Oracle and/or its affiliates. All rights reserved.

    connected to target database: SUNKDEV (DBID=3153865159)

    RMAN> BACKUP CURRENT CONTROLFILE FOR STANDBY FORMAT '/c01/standby/stdby_ctl_%U' tag 'FORSTDBY';

    Starting backup at 20-JUN-19

    using target database control file instead of recovery catalog

    allocated channel: ORA_DISK_1

    channel ORA_DISK_1: SID=42 device type=DISK

    channel ORA_DISK_1: starting full datafile backup set

    channel ORA_DISK_1: specifying datafile(s) in backup set

    including standby control file in backup set

    channel ORA_DISK_1: starting piece 1 at 20-JUN-19

    channel ORA_DISK_1: finished piece 1 at 20-JUN-19

    piece handle=/c01/standby/stdby_ctl_15u4jc1m_1_1 tag=FORSTDBY comment=NONE

    channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01

    Finished backup at 20-JUN-19

    RMAN> exit

  
6) scp the backup peices to standby server  

    [oracle@oradbdvl01 standby]$ scp -pr * oracle@orclstdy01:/c01/standby

    oracle@orclstdy01's password:

    13u4jbu3_1_1.bkp 100% 312KB 22.7MB/s 00:00

    14u4jbut_1_1.bkp 100% 9856KB 45.3MB/s 00:00

    stdby_ctl_15u4jc1m_1_1 100% 9856KB 47.6MB/s 00:00

  
7) start the standby database in nomount state  

    [oracle@orclstdy01 standby]$ sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Thu Jun 20 17:44:06 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to an idle instance.

    SQL> startup nomount

    ORACLE instance started.

    Total System Global Area 830930944 bytes

    Fixed Size 2257800 bytes

    Variable Size 524291192 bytes

    Database Buffers 301989888 bytes

    Redo Buffers 2392064 bytes

    SQL> exit

    Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, OLAP, Data Mining and Real Application Testing options

  
8)restore the controlfile from backup.  

    [oracle@orclstdy01 standby]$ rman target /

    Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jun 20 17:44:30 2019

    Copyright (c) 1982, 2011, Oracle and/or its affiliates. All rights reserved.

    connected to target database: SUNKDEV (not mounted)

    RMAN> restore standby controlfile from '/c01/standby/stdby_ctl_15u4jc1m_1_1';

    Starting restore at 20-JUN-19

    using target database control file instead of recovery catalog

    allocated channel: ORA_DISK_1

    channel ORA_DISK_1: SID=19 device type=DISK

    channel ORA_DISK_1: restoring control file

    channel ORA_DISK_1: restore complete, elapsed time: 00:00:01

    output file name=/b01/oradata/sunkdev/control01.ctl

    output file name=/c01/fra/sunkdev/control02.ctl

    Finished restore at 20-JUN-19

    RMAN> exit

    Recovery Manager complete.

  
9) bring database to mount state.  

    [oracle@orclstdy01 standby]$ !s

    sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Thu Jun 20 17:45:07 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to:

    Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, OLAP, Data Mining and Real Application Testing options

    SQL> alter database mount standby database;

    Database altered.

    SQL> exit

    Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, OLAP, Data Mining and Real Application Testing options

  
10) catalog the backup peacies.  

    [oracle@orclstdy01 standby]$ rman target /

    Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jun 20 17:45:53 2019

    Copyright (c) 1982, 2011, Oracle and/or its affiliates. All rights reserved.

    connected to target database: SUNKDEV (DBID=3153865159, not open)

    RMAN> catalog start with '/c01/standby';

    Starting implicit crosscheck backup at 20-JUN-19

    using target database control file instead of recovery catalog

    allocated channel: ORA_DISK_1

    channel ORA_DISK_1: SID=18 device type=DISK

    Crosschecked 6 objects

    Finished implicit crosscheck backup at 20-JUN-19

    Starting implicit crosscheck copy at 20-JUN-19

    using channel ORA_DISK_1

    Crosschecked 4 objects

    Finished implicit crosscheck copy at 20-JUN-19

    searching for all files in the recovery area

    cataloging files...

    cataloging done

    List of Cataloged Files

    =======================

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_16_gjpwh1gx_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_9_gjptmdcb_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_13_gjptmf5g_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_4_0tu4j954_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_31_gjpwr3od_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_17_gjpwjkvq_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_14_gjpwd72r_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_2_gjptmfot_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_26_gjpwoqqr_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_42_gjpxsr4d_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_27_gjpwoqn7_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_11_gjptmbt6_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_30_gjpwr0c6_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_8_gjptm8xv_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_3_gjptmcc6_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_7_gjptm95q_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_9_gjpvfwlh_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_8_gjpvbq7v_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_13_gjptmc2g_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_12_gjptmf04_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_7_gjpvbgkv_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_6_gjpvb91o_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_12_gjptmc12_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_7_gjptmdfb_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_28_gjpwpl87_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_33_gjpx8oyc_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_35_gjpxf0f0_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_10_gjpwh06l_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_8_gjptmd7s_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_15_gjpwh1ry_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_10_gjptmb44_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_1_gjptmfrm_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_11_gjptmdtr_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_32_gjpx8p24_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_5_gjpv855z_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_9_gjptm9gb_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_18_gjpwjqs1_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_11_gjpwh06z_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_13_gjpwh0l2_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_10_gjptmdr1_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_34_gjpxdrrx_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_12_gjpwh0b8_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_29_gjpwprxm_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_5_gj8g15fq_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_9_07u44o70_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_18_gj8cv89l_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_15_gj8cn2k6_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_25_gj8dxyb8_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_13_gj8cfrmd_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_23_gj8dsc6d_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_4_gj8fsj9x_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_10_gj8bb81s_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_21_gj8dowm6_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_11_gj8bdgtc_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_14_gj8cfvn9_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_1_gj8g31t4_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_20_gj8djxld_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_16_gj8cq54h_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_24_gj8dxk57_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_22_gj8dozdz_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_19_gj8dgncr_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_3_0eu44rlk_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_12_gj8bds0c_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_17_gj8cq81p_.arc

    File Name: /c01/fra/SUNKSBDEV/archivelog/2019_06_15/o1_mf_1_6_gj8g89bg_.arc

    searching for all files that match the pattern /c01/standby

    List of Files Unknown to the Database

    =====================================

    File Name: /c01/standby/13u4jbu3_1_1.bkp

    File Name: /c01/standby/stdby_ctl_15u4jc1m_1_1

    File Name: /c01/standby/14u4jbut_1_1.bkp

    Do you really want to catalog the above files (enter YES or NO)? yes

    cataloging files...

    cataloging done

    List of Cataloged Files

    =======================

    File Name: /c01/standby/13u4jbu3_1_1.bkp

    File Name: /c01/standby/stdby_ctl_15u4jc1m_1_1

    File Name: /c01/standby/14u4jbut_1_1.bkp

    11) recover database using rman

    RMAN> recover database;

    Starting recover at 20-JUN-19

    using channel ORA_DISK_1

    channel ORA_DISK_1: starting incremental datafile backup set restore

    channel ORA_DISK_1: specifying datafile(s) to restore from backup set

    destination for restore of datafile 00001: /b01/oradata/sunkdev/system01.dbf

    destination for restore of datafile 00002: /b01/oradata/sunkdev/sysaux01.dbf

    destination for restore of datafile 00003: /b01/oradata/sunkdev/undotbs01.dbf

    destination for restore of datafile 00004: /b01/oradata/sunkdev/users01.dbf

    destination for restore of datafile 00005: /b01/oradata/sunkdev/example01.dbf

    destination for restore of datafile 00006: /b01/oradata/sunkdev/nithints.dbf

    channel ORA_DISK_1: reading from backup piece /c01/standby/13u4jbu3_1_1.bkp

    channel ORA_DISK_1: piece handle=/c01/standby/13u4jbu3_1_1.bkp tag=TAG20190620T174123

    channel ORA_DISK_1: restored backup piece 1

    channel ORA_DISK_1: restore complete, elapsed time: 00:00:01

    starting media recovery

    archived log for thread 1 with sequence 43 is already on disk as file /c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_43_gjpy8to1_.arc

    archived log file name=/c01/fra/SUNKSBDEV/archivelog/2019_06_20/o1_mf_1_43_gjpy8to1_.arc thread=1 sequence=43

    unable to find archived log

    archived log thread=1 sequence=44

    RMAN-00571: ===========================================================

    RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============

    RMAN-00571: ===========================================================

    RMAN-03002: failure of recover command at 06/20/2019 17:46:33

    RMAN-06054: media recovery requesting unknown archived log for thread 1 with sequence 44 and starting SCN of 1115215

    RMAN> exit

  
we can safly ignore the above error as it is checking the 44th archive log file which is not generated at primary database.

12\) start the mrp process  

    [oracle@orclstdy01 standby]$ !s

    sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Thu Jun 20 17:46:36 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to:

    Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, OLAP, Data Mining and Real Application Testing options

    SQL> alter database recover managed standby database disconnect from session;

    Database altered.

  
13) perform the log switch on primary database and verify the generated archvie logs are shipped and applied at standby.  

    SQL>

    select rec.thread#, rec.last_rec "Last Sequence Received", app.last_app "Last Sequence Applied", (rec.last_rec - app.last_app) "Difference"

    from

    (select thread#, max(sequence#) last_rec from v$archived_log where resetlogs_id = (select max(resetlogs_id) from v$archived_log) group by thread#) rec,

    (select thread#, max(sequence#) last_app from v$archived_log where resetlogs_id = (select max(resetlogs_id) from v$archived_log) and applied='YES' and registrar='RFS'

    group by thread#) app

    where rec.thread#=app.thread#

    and rec.thread# != 0

    order by rec.thread#

    /SQL> 2 3 4 5 6 7 8 9

    THREAD# Last Sequence Received Last Sequence Applied Difference

    ---------- ---------------------- --------------------- ----------

    1                45                     45               0
