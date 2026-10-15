---
title: "Rman Register database with catalog"
author: "Sunke Nithin"
date: 2019-06-27
categories: ["RMAN"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/27/rman-register-database-with-catalog/
---

# Rman Register database with catalog

In this post we will see how to register oracle database to rman catalog .  

#### **Steps at catalog database**

  
**1) First create a tablespace**  

    SQL> create tablespace rmants datafile '/a01/oracle/oradata/rmandv/rmants.dbf' size 200m;

    Tablespace created.

  
2)create a user  

    SQL> create user rmanus identified by "<password>";

    User created.

  
**3)grant recovery_catalog_owner role to the user and make default tablespace rmants with unlimited quota to user rmanus.**  

    SQL> grant connect,resource,recovery_catalog_owner to rmanus;

    Grant succeeded.

    SQL> alter user rmanus default tablespace rmants;

    User altered.

    SQL> alter user rmanus quota unlimited on rmants;

    User altered.

  
**4) connect to catalog user using rman and create catalog.**  

    [oracle@oradbdvl01 scripts]$ rman catalog rmanus

    Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jun 27 21:09:37 2019 Copyright (c) 1982, 2011, Oracle and/or its affiliates. All rights reserved.

    connected to recovery catalog database

    RMAN> create catalog;

    recovery catalog created

    RMAN> exit

    Recovery Manager complete.

  

#### **steps on source database**

  
**5)  verify the tnsping to catalog database**  

    [oracle@oradbdvl01 scripts]$ tnsping rmandv

    TNS Ping Utility for Linux: Version 11.2.0.4.0 - Production on 27-JUN-2019 21:10:27

    Copyright (c) 1997, 2013, Oracle. All rights reserved.

    Used parameter files:

    Used TNSNAMES adapter to resolve the alias

    Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = oradbdvl01.snv.ind)(PORT = 1521)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = rmandv)))

    OK (10 msec)

  
**6) connect to source db and catalog db and register database.**  

    [oracle@oradbdvl01 shell]$ rman target / catalog rmanus@rmandv

    Recovery Manager: Release 11.2.0.4.0 - Production on Thu Jun 27 21:10:46 2019 Copyright (c) 1982, 2011, Oracle and/or its affiliates. All rights reserved.

    connected to target database: SUNKDEV (DBID=3153865159)

    connected to recovery catalog database

    RMAN> register database;

    database registered in recovery catalog

    starting full resync of recovery catalog

    full resync complete
