---
title: "Fast-Start-Failover-Observer"
author: "Sunke Nithin"
date: 2022-08-08
categories: ["DATAGUARD"]
tags: []
source: https://nithinsunke.wordpress.com/2022/08/08/fast-start-failover-observer/
---

# Fast-Start-Failover-Observer

``` wp-block-preformatted
1) Configure the Oracle Net settings in the $ORACLE_HOME/network/admin/tsnames.ora file:

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

``` wp-block-preformatted
2) Oracle Database version 12.2 and higher includes support for the observer to run in background mode. To do so, a wallet must be created to perform passwordless authentication to the database. Log in as the oracle user on the observer VM and perform the following tasks to create a wallet:

a.Add the following entries to $ORACLE_HOME/network/admin/sqlnet.ora:

WALLET_LOCATION =
   (SOURCE =
      (METHOD = FILE)
      (METHOD_DATA = (DIRECTORY = /h01/oracle/admin/emcdb/xdb_wallet/observer))
)
SQLNET.WALLET_OVERRIDE = TRUE

b.Create the wallet directory on the VM hosting the observer:

mkdir /h01/oracle/admin/emcdb/xdb_wallet/observer

c. Initialize the wallet, then supply a password for the wallet when prompted:

cd /h01/oracle/admin/emcdb/xdb_wallet/observer
[oracle@oem observer]$ mkstore -wrl . -create
Oracle Secret Store Tool : Version 12.2.0.1.0
Copyright (c) 2004, 2016, Oracle and/or its affiliates. All rights reserved.

Enter password:
Enter password again:
[oracle@oem observer]$ ls
cwallet.sso  cwallet.sso.lck  ewallet.p12  ewallet.p12.lck

d. Create an entry in the wallet for each database in the Data Guard configuration. The mkstore command will request the SYS password for the database, followed by the wallet password provided in the previous step:

[oracle@oem observer]$ mkstore -wrl . -createCredential npgr sys
Oracle Secret Store Tool : Version 12.2.0.1.0
Copyright (c) 2004, 2016, Oracle and/or its affiliates. All rights reserved.

Your secret/Password is missing in the command line
Enter your secret/Password:
Re-enter your secret/Password:
Enter wallet password:

[oracle@oem observer]$ mkstore -wrl . -createCredential npgrsb sys
Oracle Secret Store Tool : Version 12.2.0.1.0
Copyright (c) 2004, 2016, Oracle and/or its affiliates. All rights reserved.

Your secret/Password is missing in the command line
Enter your secret/Password:
Re-enter your secret/Password:
Enter wallet password:

e. Confirm that the wallet contains credentials for each database in the Data Guard configuration. The -listCredential argument for mkstore will request the wallet password:

[oracle@oem observer]$ mkstore -wrl . -listCredential
Oracle Secret Store Tool : Version 12.2.0.1.0
Copyright (c) 2004, 2016, Oracle and/or its affiliates. All rights reserved.

Enter wallet password:
List credential (index: connect_string username)
2: npgrsb sys
1: npgr sys
```

``` wp-block-preformatted
3. Log in to the observer VM as Oracle and then test connectivity to the primary and standby databases. Connect to the primary and secondary databases with SQL*Plus by using the wallet credentials:

[oracle@oem observer]$ sqlplus /@npgr as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Sun Aug 7 22:29:40 2022

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Last Successful login time: Sun Aug 07 2022 18:28:13 +05:30

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
[oracle@oem observer]$ sqlplus /@npgrsb as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Sun Aug 7 22:29:49 2022

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Last Successful login time: Sun Aug 07 2022 22:29:40 +05:30

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

If the wallet configuration is successful, the sqlplus command directly logs in to the database without asking for a password.
```

``` wp-block-preformatted
4. Enable Fast-Start Failover (FSFO):

ENABLE FAST_START FAILOVER;
```

``` wp-block-preformatted
[oracle@oem fsfo]$ dgmgrl
DGMGRL for Linux: Release 12.2.0.1.0 - Production on Sun Aug 7 22:30:58 2022

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Welcome to DGMGRL, type "help" for information.
DGMGRL> connect /@npgr
Connected to "NPGR"
Connected as SYSDBA.
DGMGRL> SHOW CONFIGURATION;

Configuration - my_dg_config

  Protection Mode: MaxPerformance
  Members:
  npgr   - Primary database
    npgrsb - (*) Physical standby database

Fast-Start Failover: ENABLED

Configuration Status:
SUCCESS   (status updated 9 seconds ago)
```

``` wp-block-preformatted
Enabling FSFO does not make the configuration ready for automatic failover. Automatic failover requires an observer.

Start Data Guard Observer:

DGMGRL> start observer observer2 in background logfile is '/h01/oracle/admin/fsfo/fsfo.log' connect identifier is npgr;
Submitted command "START OBSERVER" using connect identifier "npgr"
DGMGRL> show observer;

Configuration - my_dg_config

  Primary:            npgr
  Target:             npgrsb

Observer "observer1"(12.2.0.1.0) - Master

  Host Name:                    oem.snvcloud.com
  Last Ping to Primary:         0 seconds ago
  Last Ping to Target:          1 second ago
```

``` wp-block-preformatted
5) simulate the failure. on primary by doing shurt abort.

SQL> shut abort
ORACLE instance shut down.
SQL> exit
```

``` wp-block-preformatted
monitor the observer log file.

[oracle@oem fsfo]$ tail -f fsfo.log
[W000 08/07 23:11:16.92] Primary database cannot be reached.
[W000 08/07 23:11:16.92] Fast-Start Failover threshold has not exceeded. Retry for the next 60 seconds
[W000 08/07 23:11:17.93] Try to connect to the primary.
Unable to connect to database using npgr
ORA-12514: TNS:listener does not currently know of service requested in connect descriptor

[W000 08/07 23:12:16.96] Primary database cannot be reached.
[W000 08/07 23:12:16.96] Fast-Start Failover threshold has expired.
[W000 08/07 23:12:16.96] Try to connect to the standby.
[W000 08/07 23:12:16.96] Making a last connection attempt to primary database before proceeding with Fast-Start Failover.
[W000 08/07 23:12:16.97] Check if the standby is ready for failover.
Unable to connect to database using npgr
ORA-12514: TNS:listener does not currently know of service requested in connect descriptor

[S002 08/07 23:12:16.99] Fast-Start Failover started...

23:12:16.99  Sunday, August 07, 2022
Initiating Fast-Start Failover to database "npgrsb"...
[S002 08/07 23:12:17.00] Initiating Fast-start Failover.
Performing failover NOW, please wait...
Failover succeeded, new primary is "npgrsb"
23:12:34.42  Sunday, August 07, 2022
[S002 08/07 23:12:34.42] Fast-Start Failover finished...

[W000 08/07 23:12:34.42] Failover succeeded. Restart pinging.
[W000 08/07 23:12:34.46] Primary database has changed to npgrsb.
[W000 08/07 23:12:34.59] Try to connect to the primary.

now bring the new standby(npgr) to mount state.
SQL> startup mount
ORACLE instance started.

Total System Global Area 3137336128 bytes
Fixed Size                  8901440 bytes
Variable Size             671088640 bytes
Database Buffers         2449473536 bytes
Redo Buffers                7872512 bytes
Database mounted.
SQL>

[W000 08/07 23:14:07.56] Try to connect to the new standby npgr.
[W000 08/07 23:14:09.57] Connection to the new standby restored!
[W000 08/07 23:14:41.82] Try to connect to the primary npgrsb.
[W000 08/07 23:14:43.83] Connection to the primary restored!
[W000 08/07 23:14:43.83] Wait for new primary to be ready to reinstate.
[W000 08/07 23:14:43.84] New primary is now ready to reinstate.
[W000 08/07 23:14:44.84] Issuing REINSTATE command.

23:14:44.84  Sunday, August 07, 2022
Initiating reinstatement for database "npgr"...
Reinstating database "npgr", please wait...
[W000 08/07 23:15:13.95] The standby npgr is ready to be a FSFO target
Reinstatement of database "npgr" succeeded
23:15:26.94  Sunday, August 07, 2022
[W000 08/07 23:15:27.50] Successfully reinstated database npgr.
[W000 08/07 23:15:29.51] The reinstatement of standby npgr was just done
```
