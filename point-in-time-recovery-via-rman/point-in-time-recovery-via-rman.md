---
title: "POINT IN TIME RECOVERY via RMAN"
author: "Sunke Nithin"
date: 2019-07-04
categories: ["RMAN"]
tags: []
source: https://nithinsunke.wordpress.com/2019/07/04/point-in-time-recovery-via-rman/
---

# POINT IN TIME RECOVERY via RMAN

***Scenario:***

DBA dropped the tablespace TEST which is important for application.

***Requirement:***

We need to restore the database before the TEST tablespace was dropped.

***Solution:***

*SQL\> drop tablespace test including contents and datafiles;*

*Tablespace dropped.*

**DBA realized the mistake...**

**He will refer alert log for the exact timing when tablespace was dropped.**

***Alert log***

***Sun Feb  4 10:59:43 2007***

*drop tablespace test including contents and datafiles*

*Sun Feb 4 10:59:47 2007*

*Completed: drop tablespace test including contents and datafiles*

 

*SQL\> shutdown abort*

*ORACLE instance shut down.*

 

*rman target / catalog rman/cat@risl64*

*Recovery Manager: Release 10.2.0.2.0 - Production on Sun Feb 4 11:02:48 2007*

*Copyright (c) 1982, 2005, Oracle.  All rights reserved.*

*Connected to target database (not started)*

*Connected to recovery catalog database*

 

***RMAN\> run***

***{***

***startup nomount***

***set until time "to_date ('04-02-07 10:58:00', 'DD-MM-YY HH24:MI:SS')";***

***restore controlfile;***

***alter database mount;***

***restore database;***

***recover database;***

***alter database open resetlogs;***

***}***

 

*Oracle instance started*

*Total System Global Area     268435456 bytes*

*Fixed Size                     2070448 bytes*

*Variable Size                104859728 bytes*

*Database Buffers             155189248 bytes*

*Redo Buffers                   6316032 bytes*

* *

*executing command: SET until clause*

* Starting restore at 04-FEB-07*

*allocated channel: ORA_DISK_1*

*channel ORA_DISK_1: sid=157 devtype=DISK*

* channel ORA_DISK_1: starting datafile backupset restore*

*channel ORA_DISK_1: restoring control file*

*channel ORA_DISK_1: reading from backup piece /u01/ORACLE/ocm/c-1996161442-20070204-04*

*channel ORA_DISK_1: restored backup piece 1*

*piece handle=/u01/ORACLE/ocm/c-1996161442-20070204-04 tag=TAG20070204T105219*

*channel ORA_DISK_1: restore complete, elapsed time: 00:00:03*

*output filename=/u01/ORACLE/ocm/control01.ctl*

*output filename=/u01/ORACLE/ocm/control02.ctl*

*output filename=/u01/ORACLE/ocm/control03.ctl*

*Finished restore at 04-FEB-07*

* *

*database mounted*

*released channel: ORA_DISK_1*

* *

*Starting restore at 04-FEB-07*

*Starting implicit crosscheck backup at 04-FEB-07*

*allocated channel: ORA_DISK_1*

*channel ORA_DISK_1: sid=157 devtype=DISK*

*Crosschecked 18 objects*

*Finished implicit crosscheck backup at 04-FEB-07*

* *

*Starting implicit crosscheck copy at 04-FEB-07*

*using channel ORA_DISK_1*

*Finished implicit crosscheck copy at 04-FEB-07*

* *

*searching for all files in the recovery area*

*cataloging files...*

*no files cataloged*

* using channel ORA_DISK_1*

* channel ORA_DISK_1: starting datafile backupset restore*

*channel ORA_DISK_1: specifying datafile(s) to restore from backup set*

*restoring datafile 00001 to /u01/ORACLE/ocm/system01.dbf*

*restoring datafile 00002 to /u01/ORACLE/ocm/undotbs01.dbf*

*restoring datafile 00003 to /u01/ORACLE/ocm/sysaux01.dbf*

*restoring datafile 00004 to /u01/ORACLE/ocm/users01.dbf*

*restoring datafile 00005 to /u01/ORACLE/ocm/test.dbf*

*channel ORA_DISK_1: reading from backup piece /u01/ORACLE/ocm/1ni975e6_1_1*

*channel ORA_DISK_1: restored backup piece 1*

*piece handle=/u01/ORACLE/ocm/1ni975e6_1_1 tag=TAG20070204T105150*

*channel ORA_DISK_1: restore complete, elapsed time: 00:00:17*

*Finished restore at 04-FEB-07*

* *

*Starting recover at 04-FEB-07*

*using channel ORA_DISK_1*

* *

*starting media recovery*

* *

*archive log thread 1 sequence 19 is already on disk as file /u01/ORACLE/ocm/OCM/archivelog/2007_02_04/o1_mf_1_19_2wc0nztc\_.arc*

*archive log thread 1 sequence 20 is already on disk as file /u01/ORACLE/ocm/redo02.log*

*archive log filename=/u01/ORACLE/ocm/OCM/archivelog/2007_02_04/o1_mf_1_19_2wc0nztc\_.arc thread=1 sequence=19*

*archive log filename=/u01/ORACLE/ocm/redo02.log thread=1 sequence=20*

*media recovery complete, elapsed time: 00:00:00*

*Finished recover at 04-FEB-07*

* *

*database opened*

*new incarnation of database registered in recovery catalog*

*starting full resync of recovery catalog*

*full resync complete*

* *

*RMAN\>*

 

*SQL\> select name from v\$tablespace;*

* *

*NAME*

*------------------------------*

*SYSTEM*

*UNDOTBS1*

*SYSAUX*

*USERS*

*TEMP*

***TEST***

* *

*6 rows selected.*
