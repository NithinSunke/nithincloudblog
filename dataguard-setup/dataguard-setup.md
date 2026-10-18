---
title: "dataguard setup"
author: "Sunke Nithin"
date: 2022-08-08
categories: ["DATAGUARD"]
tags: []
source: https://nithinsunke.wordpress.com/2022/08/08/dataguard-setup/
---

# dataguard setup

``` wp-block-preformatted
Assumptions
• You have two servers (physical or VMs) with an operating system and Oracle installed on them. In this case I've used RHEL 7 and Oracle Database 19.14.0.0.0.
• The primary server has a running instance.
• The standby server has a software only installation.
```

``` wp-block-preformatted
Primary Server Setup
-------------------------

Check that the primary database is in archivelog mode.

SELECT log_mode FROM v$database;
LOG_MODE
------------
ARCHIVELOG
```

``` wp-block-preformatted
Enabled forced logging by issuing the following command.
ALTER DATABASE FORCE LOGGING;
```

``` wp-block-preformatted
Initialization Parameters
Check the setting for the DB_NAME and DB_UNIQUE_NAME parameters. In this case they are both set to "NPGR" on the primary database.
SQL> show parameter db_name
NAME                     TYPE    VALUE
------------------------------------ ----------- ------------------------------
db_name                  string  NPGR

SQL> show parameter db_unique_name
NAME                     TYPE    VALUE
------------------------------------ ----------- ------------------------------
db_unique_name               string  NPGR
SQL>

The DB_NAME of the standby database will be the same as that of the primary, but it must have a different DB_UNIQUE_NAME value. 
The DB_UNIQUE_NAME values of the primary and standby database should be used in the DG_CONFIG setting of the LOG_ARCHIVE_CONFIG parameter. 

ALTER SYSTEM SET LOG_ARCHIVE_CONFIG='DG_CONFIG=(NPGR,NPGRSB)';

Set suitable remote archive log destinations. 
In this case I'm using the fast recovery area for the local location, but you could specify an location explicitly if you prefer. 
Notice the SERVICE and the DB_UNIQUE_NAME for the remote location reference the standby location.

ALTER SYSTEM SET LOG_ARCHIVE_DEST_2='SERVICE=NPGRSB NOAFFIRM ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=NPGRSB';
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=ENABLE;

The LOG_ARCHIVE_FORMAT and LOG_ARCHIVE_MAX_PROCESSES parameters must be set to appropriate values and the REMOTE_LOGIN_PASSWORDFILE must be set to exclusive.

ALTER SYSTEM SET LOG_ARCHIVE_FORMAT='%t_%s_%r.arc' SCOPE=SPFILE;
ALTER SYSTEM SET LOG_ARCHIVE_MAX_PROCESSES=30;
ALTER SYSTEM SET REMOTE_LOGIN_PASSWORDFILE=EXCLUSIVE SCOPE=SPFILE;

In addition to the previous setting, it is recommended to make sure the primary is ready to switch roles to become a standby. For that to work properly we need to set the following parameters. Adjust the *_CONVERT parameters to account for your filename and path differences between the servers.

ALTER SYSTEM SET FAL_SERVER=NPGRSB;
ALTER SYSTEM SET FAL_CLIENT=NPGR;

--ALTER SYSTEM SET DB_FILE_NAME_CONVERT='NPGR','NPGRSB' SCOPE=SPFILE;
--ALTER SYSTEM SET LOG_FILE_NAME_CONVERT='NPGR','NPGRSB'  SCOPE=SPFILE;
ALTER SYSTEM SET STANDBY_FILE_MANAGEMENT=AUTO;

Remember, some of the parameters are not modifiable, so the database will need to be restarted before they take effect.
```

``` wp-block-preformatted
Entries for the primary and standby databases are needed in the "$ORACLE_HOME/network/admin/tnsnames.ora" files on both servers. 
You can create these using the Network Configuration Utility (netca) or manually. The following entries were used during this setup.

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
```

``` wp-block-code
Standby Server Setup (DUPLICATE)
Copy Files
Create the necessary directories on the standby server.
$ mkdir -p /u01/app/oracle/oradata/NPGR
$ mkdir -p /u01/app/oracle/fast_recovery_area/NPGR
$ mkdir -p /u01/app/oracle/admin/NPGR/adump
Copy the files from the primary to the standby server.

Start Listener
When using active duplicate, the standby server requires static listener configuration in a "listener.ora" file. In this case I used the following configuration.
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orstsbdvl01.snvcloud.com)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = NPGRSB_DGMGRL)
      (ORACLE_HOME = /h01/oracle/19cr3)
      (SID_NAME = NPGRSB)
    )
  )

ADR_BASE_LISTENER = /h01/oracle

update global_dbname= NPGR_DGMGRL in primary server

Make sure the listener is started on the standby server.
$ lsnrctl start
```

``` wp-block-preformatted
Create Standby Redo Logs on Primary Server
The DUPLICATE command automatically creates the standby redo logs on the standby. To make sure the primary database is configured for switchover, we must create the standby redo logs on the primary server.
ALTER DATABASE ADD STANDBY LOGFILE ('/u01/app/oracle/oradata/NPGR/standby_redo01.log') SIZE 50M;
ALTER DATABASE ADD STANDBY LOGFILE ('/u01/app/oracle/oradata/NPGR/standby_redo02.log') SIZE 50M;
ALTER DATABASE ADD STANDBY LOGFILE ('/u01/app/oracle/oradata/NPGR/standby_redo03.log') SIZE 50M;
ALTER DATABASE ADD STANDBY LOGFILE ('/u01/app/oracle/oradata/NPGR/standby_redo04.log') SIZE 50M;
```

``` wp-block-preformatted
Create Standby Using DUPLICATE

Start the auxillary instance on the standby server by starting it using the temporary "init.ora" file.
$ export ORACLE_SID=NPGRSB
[oracle@orstsbdvl01 dbs]$ cat initNPGRSB.ora
DB_NAME=NPGRSB

create the directory structure as primary

$ sqlplus / as sysdba
SQL> STARTUP NOMOUNT;
ORACLE instance started.

Total System Global Area  343932616 bytes
Fixed Size                  8896200 bytes
Variable Size             260046848 bytes
Database Buffers           67108864 bytes
Redo Buffers                7880704 bytes
SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.14.0.0.0

create/copy a password file from primary database

Connect to RMAN, specifying a full connect string for both the TARGET and AUXILLARY instances. DO not attempt to use OS authentication.

$ rman TARGET sys@npgr AUXILIARY sys@npgrsb
Now issue the following DUPLICATE command.
DUPLICATE TARGET DATABASE
  FOR STANDBY
  FROM ACTIVE DATABASE
  DORECOVER
  SPFILE
    SET db_unique_name='NPGRSB' COMMENT 'Is standby'
    SET LOG_ARCHIVE_DEST_2='SERVICE=NPGR ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=NPGR'
    SET FAL_SERVER='DB11G' COMMENT 'Is primary'
  NOFILENAMECHECK;
```

``` wp-block-preformatted
Start Apply Process
Start the apply process on standby server.
# Background redo apply. Control is returned to the session once the apply process is started.
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT FROM SESSION;
If you need to cancel the apply process, issue the following command.
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
```

``` wp-block-preformatted
Test Log Transport
On the primary server, check the latest archived redo log and force a log switch.

ALTER SESSION SET nls_date_format='DD-MON-YYYY HH24:MI:SS';
SELECT sequence#, first_time, next_time
FROM   v$archived_log
ORDER BY sequence#;
ALTER SYSTEM SWITCH LOGFILE;

Check the new archived redo log has arrived at the standby server and been applied.
ALTER SESSION SET nls_date_format='DD-MON-YYYY HH24:MI:SS';
SELECT sequence#, first_time, next_time, applied
FROM   v$archived_log
ORDER BY sequence#;
```
