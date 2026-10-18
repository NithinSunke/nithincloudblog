---
title: "configure dataguard broker on different server"
author: "Sunke Nithin"
date: 2022-08-08
categories: ["DATAGUARD"]
tags: []
source: https://nithinsunke.wordpress.com/2022/08/08/configure-dataguard-broker-on-different-server/
---

# configure dataguard broker on different server

``` wp-block-preformatted
For datagurad setup look at my previous post
https://nithinsunke.wordpress.com/2022/08/08/dataguard-setup/

1) on the seperate HOST copy/add the tnsnames of both primay and standby db_name

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
Enable Broker
At this point we have a primary database and a standby database, so now we need to start using the Data Guard Broker to manage them. 
Connect to both databases (primary and standby) and issue the following command.

ALTER SYSTEM SET dg_broker_start=true;
```

``` wp-block-preformatted
On the primary server, issue the following command to register the primary server with the broker. connect from broker mechine.

$ dgmgrl sys@NPGR
Connected to "NPGR"
Connected as SYSDBA.
DGMGRL> CREATE CONFIGURATION my_dg_config AS PRIMARY DATABASE IS npgr CONNECT IDENTIFIER IS npgr;
Configuration "my_dg_config" created with primary database "npgr"

Now add the standby database.

DGMGRL> ADD DATABASE npgrsb AS CONNECT IDENTIFIER IS npgrsb MAINTAINED AS PHYSICAL;
Database "npgrsb" added

Now we enable the new configuration.

DGMGRL> ENABLE CONFIGURATION;
Enabled.

DGMGRL>
```

``` wp-block-preformatted
The following commands show how to check the configuration and status of the databases from the broker.

DGMGRL> show configuration;

Configuration - my_dg_config

  Protection Mode: MaxPerformance
  Members:
  npgr   - Primary database
    npgrsb - Physical standby database

Fast-Start Failover:  Disabled

Configuration Status:
SUCCESS   (status updated 47 seconds ago)

DGMGRL>SHOW DATABASE NPGR;

Database - npgr

  Role:               PRIMARY
  Intended State:     TRANSPORT-ON
  Instance(s):
    NPGR

Database Status:
SUCCESS

DGMGRL> SHOW DATABASE NPGRSB;

Database - npgrsb

  Role:               PHYSICAL STANDBY
  Intended State:     APPLY-ON
  Transport Lag:      0 seconds (computed 9 seconds ago)
  Apply Lag:          0 seconds (computed 9 seconds ago)
  Average Apply Rate: 833.00 KByte/s
  Real Time Query:    ON
  Instance(s):
    NPGRSB

Database Status:
SUCCESS
```

``` wp-block-preformatted
```
