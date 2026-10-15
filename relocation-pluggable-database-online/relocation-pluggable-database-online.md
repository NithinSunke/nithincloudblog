---
title: "Relocation pluggable database online."
author: "Sunke Nithin"
date: 2019-09-05
categories: ["DATABASE"]
tags: []
source: https://nithinsunke.wordpress.com/2019/09/05/relocation-pluggable-database-online/
---

# Relocation pluggable database online.

In this post i will share how to relocate the pluggable database from one location to another location.

**considerations:**  
1) Database is in archive log mode.

**Steps:**

**1) take backup of pluggable database as copy.**  

    RMAN> backup as copy pluggable database pdbsnv format '/u01/oracle/oradata/snvtst/psnv/%U';

    Starting backup at 06-SEP-19

    allocated channel: ORA_DISK_1

    channel ORA_DISK_1: SID=49 device type=DISK

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00010 name=/u01/oracle/oradata/snvtst/pdbsnv/sysaux01.dbf

    output file name=/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-SYSAUX_FNO-10_01ub1tvq tag=TAG20190906T015258 RECID=4 STAMP=1018230785

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:15

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00009 name=/u01/oracle/oradata/snvtst/pdbsnv/system01.dbf

    output file name=/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-SYSTEM_FNO-9_02ub1u09 tag=TAG20190906T015258 RECID=5 STAMP=1018230800

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:15

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00011 name=/u01/oracle/oradata/snvtst/pdbsnv/undotbs01.dbf

    output file name=/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-UNDOTBS1_FNO-11_03ub1u0o tag=TAG20190906T015258 RECID=6 STAMP=1018230811

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:03

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00013 name=/u01/oracle/oradata/snvtst/pdbsnv/nithints.dbf

    output file name=/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-NITHINTS_FNO-13_04ub1u0s tag=TAG20190906T015258 RECID=7 STAMP=1018230812

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:01

    channel ORA_DISK_1: starting datafile copy

    input datafile file number=00012 name=/u01/oracle/oradata/snvtst/pdbsnv/users01.dbf

    output file name=/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-USERS_FNO-12_05ub1u0t tag=TAG20190906T015258 RECID=8 STAMP=1018230813

    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:01

    Finished backup at 06-SEP-19

    Starting Control File and SPFILE Autobackup at 06-SEP-19

    piece handle=/u01/oracle/12.2.0/db/dbs/c-2798789440-20190906-00 comment=NONE

    Finished Control File and SPFILE Autobackup at 06-SEP-19

  
**2) close the pluggable database**  

    RMAN> alter pluggable database pdbsnv close;

  
Statement processed  
**3) switch the pluggable database to copy.**  

    RMAN> switch pluggable database pdbsnv to copy;

    datafile 9 switched to datafile copy "/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-SYSTEM_FNO-9_02ub1u09"

    datafile 10 switched to datafile copy "/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-SYSAUX_FNO-10_01ub1tvq"

    datafile 11 switched to datafile copy "/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-UNDOTBS1_FNO-11_03ub1u0o"

    datafile 12 switched to datafile copy "/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-USERS_FNO-12_05ub1u0t"

    datafile 13 switched to datafile copy "/u01/oracle/oradata/snvtst/psnv/data_D-SNVTST_I-2798789440_TS-NITHINTS_FNO-13_04ub1u0s"

    4) recover the pluggable database with the generated archives.

    RMAN> recover pluggable database pdbsnv;

    Starting recover at 06-SEP-19

    allocated channel: ORA_DISK_1

    channel ORA_DISK_1: SID=49 device type=DISK

    starting media recovery

    archived log for thread 1 with sequence 4 is already on disk as file /u01/oracle/12.2.0/db/dbs/arch/1_4_1018226114.dbf

    archived log for thread 1 with sequence 5 is already on disk as file /u01/oracle/12.2.0/db/dbs/arch/1_5_1018226114.dbf

    archived log for thread 1 with sequence 6 is already on disk as file /u01/oracle/12.2.0/db/dbs/arch/1_6_1018226114.dbf

    archived log for thread 1 with sequence 7 is already on disk as file /u01/oracle/12.2.0/db/dbs/arch/1_7_1018226114.dbf

    archived log for thread 1 with sequence 8 is already on disk as file /u01/oracle/12.2.0/db/dbs/arch/1_8_1018226114.dbf

    archived log for thread 1 with sequence 9 is already on disk as file /u01/oracle/12.2.0/db/dbs/arch/1_9_1018226114.dbf

    archived log file name=/u01/oracle/12.2.0/db/dbs/arch/1_4_1018226114.dbf thread=1 sequence=4

    archived log file name=/u01/oracle/12.2.0/db/dbs/arch/1_5_1018226114.dbf thread=1 sequence=5

    archived log file name=/u01/oracle/12.2.0/db/dbs/arch/1_6_1018226114.dbf thread=1 sequence=6

    archived log file name=/u01/oracle/12.2.0/db/dbs/arch/1_7_1018226114.dbf thread=1 sequence=7

    media recovery complete, elapsed time: 00:00:01

    Finished recover at 06-SEP-19

  
**5) open the pluggable database.**  

    RMAN> alter pluggable database pdbsnv open;

  
Hope you like this. :)
