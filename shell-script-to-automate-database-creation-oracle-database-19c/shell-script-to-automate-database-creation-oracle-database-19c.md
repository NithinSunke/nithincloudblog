---
title: "Shell Script to automate database creation Oracle Database 19c"
author: "Sunke Nithin"
date: 2025-08-16
categories: ["ORACLE", "SHELL SCRIPTS"]
tags: ["databases", "devops", "oci", "oracle-database"]
source: https://nithinsunke.wordpress.com/2025/08/16/shell-script-to-automate-database-creation-oracle-database-19c/
---

# Shell Script to automate database creation Oracle Database 19c

Let’s make an **automation script** to create an Oracle **19c database in silent mode**.

- Take input parameters (SID, DB name, passwords, etc.).
- Run **DBCA** silently.
- Support **CDB or non-CDB** based on a flag.

#### create_db.sh

``` wp-block-code
#!/bin/bash
#
# Script: create_db.sh
# Purpose: Automate Oracle 19c database creation in silent mode
# Author: Nithin Sunke (adapted for automation)
#

# ========= CONFIGURABLE VARIABLES =========
ORACLE_BASE=/u01/app/oracle
ORACLE_HOME=$ORACLE_BASE/product/19.0.0/dbhome_1
PATH=$ORACLE_HOME/bin:$PATH

# Database configuration
DB_NAME=orcl19c         # Global Database Name
DB_SID=orcl19c          # SID
CDB_FLAG=false          # true = Create CDB with PDB, false = Non-CDB
PDB_NAME=pdb1           # Pluggable database name (if CDB_FLAG=true)

# Passwords
SYS_PWD=SysPassw0rd
SYSTEM_PWD=SysPassw0rd
PDBADMIN_PWD=PdbAdminPassw0rd

# Storage
DATA_DIR=$ORACLE_BASE/oradata
REDO_SIZE=200           # in MB
MEMORY_PERCENT=40       # % of available memory to allocate

# =========================================

echo "============================================="
echo "   Oracle 19c Database Silent Creation"
echo "   SID      : $DB_SID"
echo "   DB Name  : $DB_NAME"
echo "   CDB Mode : $CDB_FLAG"
echo "============================================="

# Run DBCA
if [ "$CDB_FLAG" = true ]; then
    echo "[INFO] Creating CDB with PDB..."
    dbca -silent -createDatabase \
        -templateName General_Purpose.dbc \
        -gdbName $DB_NAME \
        -sid $DB_SID \
        -createAsContainerDatabase true \
        -numberOfPDBs 1 \
        -pdbName $PDB_NAME \
        -pdbAdminPassword $PDBADMIN_PWD \
        -responseFile NO_VALUE \
        -characterSet AL32UTF8 \
        -sysPassword $SYS_PWD \
        -systemPassword $SYSTEM_PWD \
        -databaseType MULTIPURPOSE \
        -memoryPercentage $MEMORY_PERCENT \
        -storageType FS \
        -datafileDestination $DATA_DIR \
        -redoLogFileSize $REDO_SIZE \
        -emConfiguration NONE
else
    echo "[INFO] Creating Non-CDB database..."
    dbca -silent -createDatabase \
        -templateName General_Purpose.dbc \
        -gdbName $DB_NAME \
        -sid $DB_SID \
        -createAsContainerDatabase false \
        -responseFile NO_VALUE \
        -characterSet AL32UTF8 \
        -sysPassword $SYS_PWD \
        -systemPassword $SYSTEM_PWD \
        -databaseType MULTIPURPOSE \
        -memoryPercentage $MEMORY_PERCENT \
        -storageType FS \
        -datafileDestination $DATA_DIR \
        -redoLogFileSize $REDO_SIZE \
        -emConfiguration NONE
fi

# Verify result
if [ $? -eq 0 ]; then
    echo "[SUCCESS] Database $DB_NAME created successfully."
else
    echo "[ERROR] Database creation failed. Check logs in $ORACLE_BASE/cfgtoollogs/dbca/$DB_NAME"
    exit 1
fi
```

## Usage

1.  Save the script as `create_db.sh`.
2.  Make it executable: `chmod +x create_db.sh`
3.  Run it:

``` wp-block-code
./create_db.sh
```

## What This Script Does

- Creates **CDB with PDB** if `CDB_FLAG=true`.
- Creates a **non-CDB** if `CDB_FLAG=false`.
- Stores database files in `$ORACLE_BASE/oradata`.
- Runs completely **silent**, no GUI required.
