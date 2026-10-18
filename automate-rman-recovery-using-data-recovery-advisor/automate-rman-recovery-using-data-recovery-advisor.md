---
title: "Automate Rman Recovery using Data Recovery Advisor"
author: "Sunke Nithin"
date: 2019-06-20
categories: ["RMAN"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/20/automate-rman-recovery-using-data-recovery-advisor/
---

# Automate Rman Recovery using Data Recovery Advisor

Data Recovery Advisor is the new feature in oracle 11g database by which it is not only present the option but also it find issue and fix the problem. It can perform repair and recovery process at user request.

The Data recovery advisor can be used via OEM console and rman command line interface.

.

Lets look at the example as below.  
first i will create a tablespace nithin_ts and create a user nithin  

    SQL> create tablespace nithin_ts datafile '/b01/oradata/sunkdev/nithints.dbf' size 10m;

    Tablespace created.

    SQL> create user nithin identified by <password>;

    User created.

    grant privilleges to connect to database and make default tablespace nithin_ts

    SQL> grant connect,resource to nithin;

    Grant succeeded.

    SQL> alter user nithin default tablespace nithin_ts;

    User altered.

    SQL> alter user nithin quota unlimited on nithin_ts;

    User altered.

    SQL> grant select on scott.emp to nithin;

    Grant succeeded.

    SQL> conn nithin

    Connected.

    SQL> create table emp as select * from scott.emp;

    Table created.

  
2. i will take the backup of the database  

    [oracle@oradbdvl01 cmd]$ rman target / cmdfile=offline_backup.cmd

    Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jun 20 15:32:24 2019

    Copyright (c) 1982, 2011, Oracle and/or its affiliates. All rights reserved.

    connected to target database: SUNKDEV (DBID=3153865159)

    RMAN> shutdown immediate

    2> startup mount

    3> run

    4> {

    5> allocate channel c1 device type disk format '/c01/backup/%U';

    6> backup incremental level=0 database tag 'firstbackup';

    7> backup current controlfile tag 'control bkp';

    8> }

    9> alter database open;

    10>

    using target database control file instead of recovery catalog

    database closed

    database dismounted

    Oracle instance shut down

    connected to target database (not started)

    Oracle instance started

    database mounted

    Total System Global Area 830930944 bytes

    Fixed Size 2257800 bytes

    Variable Size 524291192 bytes

    Database Buffers 301989888 bytes

    Redo Buffers 2392064 bytes

    allocated channel: c1

    channel c1: SID=18 device type=DISK

    Starting backup at 20-JUN-19

    channel c1: starting incremental level 0 datafile backup set

    channel c1: specifying datafile(s) in backup set

    input datafile file number=00001 name=/b01/oradata/sunkdev/system01.dbf

    input datafile file number=00002 name=/b01/oradata/sunkdev/sysaux01.dbf

    input datafile file number=00005 name=/b01/oradata/sunkdev/example01.dbf

    input datafile file number=00003 name=/b01/oradata/sunkdev/undotbs01.dbf

    input datafile file number=00006 name=/b01/oradata/sunkdev/nithints.dbf

    input datafile file number=00004 name=/b01/oradata/sunkdev/users01.dbf

    channel c1: starting piece 1 at 20-JUN-19

    channel c1: finished piece 1 at 20-JUN-19

    piece handle=/c01/backup/0iu4j4cr_1_1 tag=FIRSTBACKUP comment=NONE

    channel c1: backup set complete, elapsed time: 00:01:15

    channel c1: starting incremental level 0 datafile backup set

    channel c1: specifying datafile(s) in backup set

    including current control file in backup set

    including current SPFILE in backup set

    channel c1: starting piece 1 at 20-JUN-19

    channel c1: finished piece 1 at 20-JUN-19

    piece handle=/c01/backup/0ju4j4f5_1_1 tag=FIRSTBACKUP comment=NONE

    channel c1: backup set complete, elapsed time: 00:00:01

    Finished backup at 20-JUN-19

    Starting backup at 20-JUN-19

    channel c1: starting full datafile backup set

    channel c1: specifying datafile(s) in backup set

    including current control file in backup set

    channel c1: starting piece 1 at 20-JUN-19

    channel c1: finished piece 1 at 20-JUN-19

    piece handle=/c01/backup/0ku4j4fe_1_1 tag=CONTROL BKP comment=NONE

    channel c1: backup set complete, elapsed time: 00:00:01

    Finished backup at 20-JUN-19

    released channel: c1

    database opened

    Recovery Manager complete.

  
3) at os level i will remove the datafile creating a situation of lost of datafile.  

    [oracle@oradbdvl01 backup]$ rm /b01/oradata/sunkdev/nithints.dbf

  
4) now as rman execute data recovery adviser  

    [oracle@oradbdvl01 backup]$ rman target /

    Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jun 20 15:43:27 2019

    Copyright (c) 1982, 2011, Oracle and/or its affiliates. All rights reserved.

    connected to target database: SUNKDEV (DBID=3153865159)

  

    RMAN> list failure;

    using target database control file instead of recovery catalog

    List of Database Failures

    =========================

    Failure ID Priority Status Time Detected Summary

    ---------- -------- --------- ------------- -------

    572 HIGH OPEN 20-JUN-19 One or more non-system datafiles are missing

  

    RMAN> advise failure;

    List of Database Failures

    =========================

    Failure ID Priority Status Time Detected Summary

    ---------- -------- --------- ------------- -------

    572 HIGH OPEN 20-JUN-19 One or more non-system datafiles are missing

    analyzing automatic repair options; this may take some time

    allocated channel: ORA_DISK_1

    channel ORA_DISK_1: SID=38 device type=DISK

    analyzing automatic repair options complete

    Mandatory Manual Actions

    ========================

    no manual actions available

    Optional Manual Actions

    =======================

    1. If file /b01/oradata/sunkdev/nithints.dbf was unintentionally renamed or moved, restore it

    Automated Repair Options

    ========================

    Option Repair Description

    ------ ------------------

    1 Restore and recover datafile 6

    Strategy: The repair includes complete media recovery with no data loss

    Repair script: /a01/oracle/diag/rdbms/sunkdev/sunkdev/hm/reco_1991144589.hm

  

    RMAN> repair failure preview;

    Strategy: The repair includes complete media recovery with no data loss

    Repair script: /a01/oracle/diag/rdbms/sunkdev/sunkdev/hm/reco_1991144589.hm

    contents of repair script:

    # restore and recover datafile

    sql 'alter database datafile 6 offline';

    restore datafile 6;

    recover datafile 6;

    sql 'alter database datafile 6 online';

  

    RMAN> repair failure;

    Strategy: The repair includes complete media recovery with no data loss

    Repair script: /a01/oracle/diag/rdbms/sunkdev/sunkdev/hm/reco_1991144589.hm

    contents of repair script:

    # restore and recover datafile

    sql 'alter database datafile 6 offline';

    restore datafile 6;

    recover datafile 6;

    sql 'alter database datafile 6 online';

    Do you really want to execute the above repair (enter YES or NO)? yes

    executing repair script

    sql statement: alter database datafile 6 offline

    Starting restore at 20-JUN-19

    using channel ORA_DISK_1

    channel ORA_DISK_1: starting datafile backup set restore

    channel ORA_DISK_1: specifying datafile(s) to restore from backup set

    channel ORA_DISK_1: restoring datafile 00006 to /b01/oradata/sunkdev/nithints.dbf

    channel ORA_DISK_1: reading from backup piece /c01/backup/0iu4j4cr_1_1

    channel ORA_DISK_1: piece handle=/c01/backup/0iu4j4cr_1_1 tag=FIRSTBACKUP

    channel ORA_DISK_1: restored backup piece 1

    channel ORA_DISK_1: restore complete, elapsed time: 00:00:01

    Finished restore at 20-JUN-19

    Starting recover at 20-JUN-19

    using channel ORA_DISK_1

    starting media recovery

    media recovery complete, elapsed time: 00:00:01

    Finished recover at 20-JUN-19

    sql statement: alter database datafile 6 online

    repair failure complete

    RMAN> exit

    Recovery Manager complete.

  
5) lastly verify at os level the datafile will be created.  

    [oracle@oradbdvl01 backup]$ ls /b01/oradata/sunkdev/nithints.dbf

    /b01/oradata/sunkdev/nithints.dbf

    [oracle@oradbdvl01 backup]$
