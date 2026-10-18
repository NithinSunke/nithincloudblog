---
title: "Clone oracle database 19c using refreshable clones"
author: "Sunke Nithin"
date: 2024-06-09
categories: ["Uncategorized"]
tags: ["clone", "DATABASE", "ORACLE", "oracle-database", "refresh"]
source: https://nithinsunke.wordpress.com/2024/06/09/clone-oracle-database-19c-using-refreshable-clones/
---

# Clone oracle database 19c using refreshable clones

In this we will see how to do clone of pluggable database using refreshable clone feature.

|                  |             |
|------------------|-------------|
| **Name**         | **Details** |
| Source CDB       | CDB1        |
| Target CDB       | crac1d      |
| Source PDB       | o122v       |
| Target PDB       | 0112t       |
| Database Version | 19c         |

#### On source :

Create a cloneuser for the refresh activity on container level

``` wp-block-code
SQL> create user c##cloneuser identified by xxxxx;

User created.

SQL> grant create session, create pluggable database,select_catalog_role to c##cloneuser;

Grant succeeded.
```

#### On target:

Add tnsnames of source container

``` wp-block-code
CDB1 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = ol7_19c_vagrant.scslabs.com)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = cdb1)
    )
  )

$ tnsping cdb1

TNS Ping Utility for Linux: Version 19.0.0.0.0 - Production on 09-JUN-2024 14:01:15

Copyright (c) 1997, 2019, Oracle.  All rights reserved.

Used parameter files:

Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = ol7_19c_vagrant.scslabs.com)(PORT = 1521)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = cdb1)))
OK (10 msec)
```

Create a database link

``` wp-block-code
SQL> create database link cloneo122v connect to c##cloneuser identified by xxxx using 'cdb1';

Database link created.

SQL> SELECT * FROM DUAL@cloneo122v;

D
-
X
```

#### Create the refreshable pluggable database using db link as below

``` wp-block-code
SQL> create pluggable database O122V from O122V@cloneo122v file_name_convert=('/u02/oradata/CDB1/18ED0CF355963BE9E0631901A8C07D5D','+DATA/CRAC1D/O122V') parallel 3 refresh mode manual;

Pluggable database created.
```

If needed perform the refresh of pdb when ever required.

``` wp-block-code
SQL> SQL> alter pluggable database O122V refresh;

Pluggable database altered.
```

To end the refresh

``` wp-block-code
SQL> alter pluggable database O122V refresh mode none;

Pluggable database altered.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 P19C1D                         READ WRITE NO
         5 O122V                          MOUNTED
SQL> SQL> alter pluggable database O122V open;

Pluggable database altered.
```

Now we rename the pdb to the desired name(O122T)

``` wp-block-code
SQL> alter pluggable database O122V close immediate;

Pluggable database altered.

SQL> alter pluggable database O122V  open restricted;

Pluggable database altered.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 P19C1D                         READ WRITE NO
         5 O122V                          READ WRITE YES
SQL> alter session set container=O122V;

Session altered.

SQL> alter pluggable database rename global_name to O122T;

Pluggable database altered.

SQL> alter pluggable database O122T close immediate;

Pluggable database altered.

SQL> alter pluggable database O122T open;

Pluggable database altered.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         5 O122T                          READ WRITE NO
SQL> conn / as sysdba
Connected.
SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 P19C1D                         READ WRITE NO
         5 O122T                          READ WRITE NO
SQL>

SQL> alter pluggable database  O122T save state;

Pluggable database altered.
```
