---
title: "Migrating Oracle 19c Non-CDB to PDB Using Export and Import"
author: "Sunke Nithin"
date: 2025-08-18
categories: ["ORACLE"]
tags: ["AI", "DATABASE", "SQL", "technology"]
source: https://nithinsunke.wordpress.com/2025/08/18/migrating-oracle-19c-non-cdb-to-pdb-using-export-and-import/
---

# Migrating Oracle 19c Non-CDB to PDB Using Export and Import

Oracle’s multitenant architecture, introduced in Oracle Database 12c and enhanced in 19c, allows multiple Pluggable Databases (PDBs) to be managed within a single Container Database (CDB), offering improved resource utilization and manageability. For Oracle 19c databases, migrating a traditional non-Container Database (non-CDB) to a PDB is a strategic move to align with modern database practices. This blog post provides a comprehensive guide to performing this migration using the export and import method with Oracle Data Pump (expdp and impdp). This approach is flexible, allowing schema or full database migration with data transformation options.

## Introduction

The export and import method leverages Oracle Data Pump to extract data from a non-CDB and import it into a PDB within a CDB. Unlike direct conversion methods, this approach does not require the source database to be in a specific state (e.g., READ ONLY).

### Step-by-Step Process

#### Step 1: Prepare the Environment

Create a directory object for Data Pump on both databases:

``` wp-block-code
sqlplus / as sysdba
CREATE OR REPLACE DIRECTORY dpump_dir AS '/u01/app/oracle/dpump';
GRANT READ, WRITE ON DIRECTORY dpump_dir TO system;
```

### Step 2: Export Data from the Non-CDB

Create a parameter file (e.g., export.par) for a full export:

``` wp-block-code
DIRECTORY=dpump_dir
DUMPFILE=noncdb_export.dmp
LOGFILE=noncdb_export.log
FULL=Y
```

Perform the export as the system user:

``` wp-block-code
expdp system/<password> PARFILE=export.par
```

Monitor the noncdb_export.log for completion and errors.

#### Step 3: Transfer the Dump File (if Cross-Host)

Use scp or a similar tool to transfer the file

Verify the file’s integrity and permissions on the target

#### Step 4: Prepare the Target PDB

Create the PDB if not already present

#### Step 5: Import Data into the PDB

Load data into the PDB using Data Pump.

Perform the import:

``` wp-block-code
impdp system/<password> PARFILE=import.par
```

#### Step 6: Post-Import Validation and Cleanup

Validate data and objects:

``` wp-block-code
SELECT COUNT(*) FROM dba_objects;
SELECT COUNT(*) FROM user_tables;
```

Recompile invalid objects:

``` wp-block-code
@?/rdbms/admin/utlrp.sql
```

Update tnsnames.ora for the PDB:

``` wp-block-code
pdb_noncdb =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = ONCDB)
    )
  )
```

- Backup the CDB:
- Optionally drop the non-CDB:

## Conclusion

Migrating an Oracle 19c non-CDB to a PDB using the export and import method with Data Pump offers a versatile and controlled approach. By following these detailed steps, you can ensure a successful transition while leveraging Data Pump’s features for data management. Always test in a non-production environment and consult the Oracle Data Pump documentation or import/export logs for troubleshooting.
