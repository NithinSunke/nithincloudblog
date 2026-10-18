---
title: "How to Create an Oracle 19c NON-CDB Database in Silent Mode"
author: "Sunke Nithin"
date: 2025-08-16
categories: ["ORACLE"]
tags: []
source: https://nithinsunke.wordpress.com/2025/08/16/how-to-create-an-oracle-19c-non-cdb-database-in-silent-mode/
---

# How to Create an Oracle 19c NON-CDB Database in Silent Mode

In many enterprise environments, database administrators (DBAs) prefer to automate Oracle installations instead of using the graphical Database Configuration Assistant (DBCA). Silent mode installation is faster, repeatable, and well-suited for scripting in Dev/Test/Prod environments.

In this post, we’ll walk through the process of creating an **Oracle Database 19c** in **silent mode** using `dbca`.

## Why Silent Mode?

- No GUI required → useful for servers without X Window support.
- Automates installations → repeatable across environments.
- Perfect for DevOps, scripting, and CI/CD pipelines.

## Prerequisites

1.  Oracle Database 19c software installed (silent or GUI).
2.  Environment variables set:

``` wp-block-code
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_2
export ORACLE_BASE=/u01/app/oracle
export PATH=$ORACLE_HOME/bin:$PATH
```

3\. Listener configured (or DBCA can create one during setup).

### Creating a Database in Silent Mode

#### Non-CDB Database

``` wp-block-code
dbca -silent -createDatabase \
        -templateName General_Purpose.dbc \
        -gdbName $DB_NAME \
        -sid ONCDB \
        -createAsContainerDatabase false \
        -responseFile NO_VALUE \
        -characterSet AL32UTF8 \
        -sysPassword xxxxx \
        -systemPassword xxxx \
        -databaseType MULTIPURPOSE \
        -memoryPercentage 40 \
        -storageType FS \
        -datafileDestination /u02/oradata \
        -redoLogFileSize 200 \
        -emConfiguration NONE
```

``` wp-block-code
Prepare for db operation
10% complete
Copying database files
40% complete
Creating and starting Oracle instance
42% complete
46% complete
50% complete
54% complete
60% complete
Completing Database Creation
66% complete
69% complete
70% complete
Executing Post Configuration Actions
100% complete
Database creation complete. For details check the logfiles at:
 /u01/app/oracle/cfgtoollogs/dbca/ONCDB.
```

## Conclusion

Creating a database in silent mode with `dbca` is a must-know skill for DBAs managing Oracle in modern environments.

- Use **`-createAsContainerDatabase false`** if you need a **non-CDB** (legacy).

With silent mode, you can script, automate, and deploy databases consistently across environments—saving time and reducing human error.
