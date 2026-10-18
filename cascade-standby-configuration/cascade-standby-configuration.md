---
title: "cascade standby configuration."
author: "Sunke Nithin"
date: 2022-08-08
categories: ["DATAGUARD"]
tags: []
source: https://nithinsunke.wordpress.com/2022/08/08/cascade-standby-configuration/
---

# cascade standby configuration.

``` wp-block-preformatted
▲
cascade standby configuration.
▲
cascade standby is create standby from another standby which will offload the load on primary database as primary logs ships the redo log to one standby database from there the redo logs is shipped to another standby database.
below are the steps of cascade standby db setup.

1) create new listener in cascade standby server.
[oracle@rac1 admin]$ cat listener.ora
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = rac1.snvcloud.com)(PORT = 1522))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1522))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = NPGRSB1_DGMGRL)
      (ORACLE_HOME = /h01/oracle/19cr3)
      (SID_NAME = NPGRSB1)
    )
  )

2) make sure we have all 3server tnsnames entries are present in all 3 servers.
NPGR =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = orstdbdvl01.snvcloud.com)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = NPGR)
    )
  )

NPGRSB =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = orstsbdvl01.snvcloud.com)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = NPGRSB)
    )
  )

NPGRSB1 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = rac1.snvcloud.com)(PORT = 1522))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = NPGRSB1)
    )
  )

3) verify tns pinging

[oracle@rac1 admin]$ tnsping npgr

TNS Ping Utility for Linux: Version 19.0.0.0.0 - Production on 08-AUG-2022 15:32:20

Copyright (c) 1997, 2019, Oracle.  All rights reserved.

Used parameter files:

Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = orstdbdvl01.snvcloud.com)(PORT = 1521)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = NPGR)))
OK (10 msec)

[oracle@rac1 admin]$ tnsping npgrsb

TNS Ping Utility for Linux: Version 19.0.0.0.0 - Production on 08-AUG-2022 15:32:21

Copyright (c) 1997, 2019, Oracle.  All rights reserved.

Used parameter files:

Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = orstsbdvl01.snvcloud.com)(PORT = 1521)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = NPGRSB)))
OK (0 msec)

[oracle@rac1 admin]$ tnsping npgrsb1

TNS Ping Utility for Linux: Version 19.0.0.0.0 - Production on 08-AUG-2022 15:32:23

Copyright (c) 1997, 2019, Oracle.  All rights reserved.

Used parameter files:

Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = rac1.snvcloud.com)(PORT = 1522)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = NPGRSB1)))
OK (0 msec)

4) Changing parameters in primary database:
ALTER SYSTEM SET db_unique_name='npgr' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_config='dg_config=(npgr,npgrsb,npgrsb1)' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_dest_2='service=npgrsb async valid_for=(online_logfiles,primary_role) db_unique_name=npgrsb' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_dest_3='service=npgrsb1 async valid_for=(standby_logfiles,standby_role) db_unique_name=npgrsb1' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_dest_state_3='DEFER' SCOPE=SPFILE;
ALTER SYSTEM SET fal_server='npgrsb' SCOPE=SPFILE;
ALTER SYSTEM SET fal_client='NPGR' SCOPE=SPFILE;
ALTER SYSTEM SET standby_file_management='AUTO' SCOPE=SPFILE;
SHUT IMMEDIATE
STARTUP

5) copy the password and pfile from standy(npgrsb) to cascade standby(npgrsb1)
[oracle@orstsbdvl01 dbs]$ scp -pr orapwNPGRSB oracle@rac1.snvcloud.com:/h01/oracle/19cr3/dbs
oracle@rac1.snvcloud.com's password:
orapwNPGRSB                                                                         100% 6144   871.4KB/s   00:00

[oracle@orstsbdvl01 dbs]$ scp -pr initNPGRSB.ora oracle@rac1.snvcloud.com:/h01/oracle/19cr3/dbs
oracle@rac1.snvcloud.com's password:
initNPGRSB.ora                                                                      100% 1459   556.7KB/s   00:00

6) Changing parameters in standby(npgrsb) database:
ALTER SYSTEM SET db_unique_name='npgrsb' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_config='dg_config=(npgr,npgrsb,npgrsb1)' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_dest_2='service=npgr async valid_for=(online_logfiles,primary_role) db_unique_name=npgr' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_dest_3='service=npgrsb1 async valid_for=(standby_logfiles,standby_role) db_unique_name=npgrsb1' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_dest_state_3='ENABLE' SCOPE=SPFILE;
ALTER SYSTEM SET fal_server='npgr' SCOPE=SPFILE;
ALTER SYSTEM SET fal_client='npgrsb' SCOPE=SPFILE;
ALTER SYSTEM SET standby_file_management='AUTO' SCOPE=SPFILE;

7) make necessary directories structure.

[oracle@rac1 dbs]$ mkdir -p /h01/oracle/admin/NPGR/adump /h01/oracle/oradata/NPGRSB1 /h01/oracle/fast_recovery_area/NPGRSB1

8) duplicate the target database from standby database.

DUPLICATE TARGET DATABASE
  FOR STANDBY
  FROM ACTIVE DATABASE
  DORECOVER
  spfile
    SET db_unique_name='npgrsb1' COMMENT 'Is standby'
    SET control_files='/h01/oracle/oradata/NPGRSB1/control01.ctl','/h01/oracle/fast_recovery_area/NPGRSB1/control02.ctl'
    SET LOG_ARCHIVE_DEST_2='SERVICE=npgrsb ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=npgrsb'
    set log_archive_dest_3='service=npgrsb1 async valid_for=(standby_logfiles,standby_role) db_unique_name=npgrsb1'
    set log_archive_dest_state_3='ENABLE'
    set audit_file_dest='/h01/oracle/admin/NPGRSB1/adump'
    set db_create_file_dest='/h01/oracle/oradata/NPGRSB1'
    set db_recovery_file_dest='/h01/oracle/fast_recovery_area'
    SET FAL_SERVER='npgrsb' COMMENT 'Is standby1'
    set fal_client='npgrsb1'
  NOFILENAMECHECK;

[oracle@rac1 dbs]$ dgmgrl sys@npgr
DGMGRL for Linux: Release 19.0.0.0.0 - Production on Mon Aug 8 19:15:44 2022
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Welcome to DGMGRL, type "help" for information.
Connected to "NPGR"
Connected as SYSDBA.
DGMGRL> add database npgrsb1 as connect identifier is npgrsb1 maintained as physical;
Database "npgrsb1" added
DGMGRL> show configuration;

Configuration - my_dg_config

  Protection Mode: MaxPerformance
  Members:
  npgr    - Primary database
    Error: ORA-16810: multiple errors or warnings detected for the member

    npgrsb  - (*) Physical standby database
      Warning: ORA-16829: fast-start failover configuration is lagging

    npgrsb1 - Physical standby database (disabled)
      ORA-16905: The member was not enabled yet.

Fast-Start Failover: Enabled in Potential Data Loss Mode

Configuration Status:
ERROR   (status updated 41 seconds ago)

--it you are using rac or Asm update below entry in grid home listener.

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = NPGRSB1_DGMGRL)
      (ORACLE_HOME = /h01/oracle/19cr3)
      (SID_NAME = NPGRSB1)
    )
  )

--enable configuration
DGMGRL> enable database 'npgrsb1';
Enabled.

DGMGRL> show configuration;

Configuration - my_dg_config

  Protection Mode: MaxPerformance
  Members:
  npgr    - Primary database
    npgrsb  - (*) Physical standby database
    npgrsb1 - Physical standby database

Fast-Start Failover: ENABLED

Configuration Status:
SUCCESS   (status updated 53 seconds ago)
```
