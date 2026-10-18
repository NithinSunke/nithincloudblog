---
title: "converting single instance asm database  to rac instance"
author: "Sunke Nithin"
date: 2019-06-24
categories: ["RAC"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/24/converting-single-instance-asm-database-to-rac-instance/
---

# converting single instance asm database  to rac instance

a\) **create the redo log groups for thread 2**  

    SQL> alter database add logfile thread 2 group 4 '+data';

    Database altered.

    SQL> alter database add logfile thread 2 group 5 '+data';

    Database altered.

    SQL> alter database add logfile thread 2 group 6 '+data';

    select

    Database altered.

    SQL> member from v$logfile;

    MEMBER

    --------------------------------------------------------------------------------

    +DATA/asmrac/onlinelog/group_4.282.1011822639

    +DATA/asmrac/onlinelog/group_5.283.1011822651

    +DATA/asmrac/onlinelog/group_6.284.1011822661

    +DATA/asmrac/onlinelog/group_1.279.1011816477

    +DATA/asmrac/onlinelog/group_2.280.1011816489

    +DATA/asmrac/onlinelog/group_3.281.1011816497

    6 rows selected.

  
b) **now enable thread 2.**  

    SQL> alter database enable thread 2;

    Database altered.

  
c) **create undo tablespace from 2nd instance**  

    SQL> show parameter undo;

    NAME TYPE VALUE

    ------------------------------------ ----------- ------------------------------

    undo_management string AUTO

    undo_retention integer 900

    undo_tablespace string UNDOTBS1

    SQL> create undo tablespace undotbs2 datafile '+data' size 200m;

    Tablespace created.

  
d) **add cluster releated parameter**  

    cluster_database=true

    cluster_database_instances=2

    asmrac1.instance_number=1

    asmrac2.instance_number=2

    asmrac1.undo_tablespace='UNDOTBS1'

    asmrac2.undo_tablespace='UNDOTBS2'

  
create instance specific parmaeter files  

    [oracle@rac1 dbs]$ cp initasmrac.ora initasmrac1.ora

    [oracle@rac1 dbs]$ cp initasmrac.ora initasmrac2.ora

  
copy it to secound node  

    [oracle@rac1 dbs]$ scp -pr initasmrac2.ora oracle@rac2:/o01/home/oracle/11.2.0.4/db/dbs

    initasmrac2.ora 100% 1073 768.4KB/s 00:00

  
e) **shutdown the database**  

    [oracle@rac1 dbs]$ sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Mon Jun 24 22:02:11 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to:

    Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,

    Data Mining and Real Application Testing options

    SQL> shut immediate

    Database closed.

    Database dismounted.

    ORACLE instance shut down.

    SQL> exit

    Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,

    Data Mining and Real Application Testing options

  
f) **start one by one instance**  

    node1

    [oracle@rac1 dbs]$ export ORACLE_SID=asmrac1

    [oracle@rac1 dbs]$ sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Mon Jun 24 22:03:03 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to an idle instance.

    SQL> startup nomount pfile='initasmrac1.ora'

    ORACLE instance started.

    Total System Global Area 1023004672 bytes

    Fixed Size 2259640 bytes

    Variable Size 704644424 bytes

    Database Buffers 310378496 bytes

    Redo Buffers 5722112 bytes

    SQL> alter database mount;

    Database altered.

    SQL> alter database open;

    Database altered.

    SQL> exit

  
**node2**  

    before starting create audit file location directory

    [oracle@rac1 dbs]$ export ORACLE_SID=asmrac2

    [oracle@rac1 dbs]$ sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Mon Jun 24 22:03:03 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to an idle instance.

    SQL> startup nomount pfile='initasmrac2.ora'

    ORACLE instance started.

    Total System Global Area 1023004672 bytes

    Fixed Size 2259640 bytes

    Variable Size 704644424 bytes

    Database Buffers 310378496 bytes

    Redo Buffers 5722112 bytes

    SQL> alter database mount;

    Database altered.

    SQL> alter database open;

    Database altered.

    SQL> exit

  
g) **stop complete database**  

    [oracle@rac1 ~]$ sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Mon Jun 24 22:14:26 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to:

    Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,

    Data Mining and Real Application Testing options

    SQL> shut immediate

    Database closed.

    Database dismounted.

    ORACLE instance shut down.

    SQL> exit

    [oracle@rac2 ~]$ sqlplus / as sysdba

    SQL*Plus: Release 11.2.0.4.0 Production on Mon Jun 24 22:14:26 2019

    Copyright (c) 1982, 2013, Oracle. All rights reserved.

    Connected to:

    Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

    With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,

    Data Mining and Real Application Testing options

    SQL> shut immediate

    Database closed.

    Database dismounted.

    ORACLE instance shut down.

    SQL> exit

  
 

**h) register database into cluster**  

    [oracle@rac1 dbs]$ srvctl remove database -d asmrac

    Remove the database asmrac? (y/[n]) y

    [oracle@rac1 dbs]$ vi /etc/oratab

    [oracle@rac1 dbs]$ srvctl add database -d asmrac -o /o01/home/oracle/11.2.0.4/db

    [oracle@rac1 dbs]$ srvctl add instance -d asmrac -i asmrac1 -n rac1

    [oracle@rac1 dbs]$ srvctl add instance -d asmrac -i asmrac2 -n rac2

    [oracle@rac1 dbs]$ srvctl start database -d asmrac

    [oracle@rac1 dbs]$ srvctl status database -d asmrac

    Instance asmrac1 is running on node rac1

    Instance asmrac2 is running on node rac2

    i) create spfile to asm diskgroup

    SQL> create spfile='+DATA' FROM PFILE;

    File created.

    [oracle@rac1 dbs]$ cat initasmrac1.ora

    SPFILE='+data/asmrac/PARAMETERFILE'
