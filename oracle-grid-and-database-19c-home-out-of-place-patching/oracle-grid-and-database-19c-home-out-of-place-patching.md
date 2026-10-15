---
title: "Oracle Grid and database 19c home out-of-place patching"
author: "Sunke Nithin"
date: 2024-06-11
categories: ["Uncategorized"]
tags: ["DATABASE", "ORACLE", "ORACLE CLOUD", "oracle-database"]
source: https://nithinsunke.wordpress.com/2024/06/11/oracle-grid-and-database-19c-home-out-of-place-patching/
---

# Oracle Grid and database 19c home out-of-place patching

In this blog we will see how to perform out-of-place patching. This method is useful when the grid/db home mount points are got full and unable to extend the file system and want to move to other mount point where it has sufficient space.

Supported versions : Oracle Database - Enterprise Edition - Version 12.2.0.1 and later

### Assumptions:

1.  Oracle standalone db restart is already configured with 19c version
2.  Download all the latest RU patches and latest OPatch from MOS

Steps:

``` wp-block-code
[root@ol7asmdb01 patch]# $ORACLE_HOME/OPatch/opatchauto apply /u01/patch/36233126 -outofplace

OPatchauto session is initiated at Mon Jun 10 15:55:42 2024

System initialization log file is /u01/app/19.0.0/grid/cfgtoollogs/opatchautodb/systemconfig2024-06-10_03-55-53PM.log.

Session log file is /u01/app/19.0.0/grid/cfgtoollogs/opatchauto/opatchauto2024-06-10_03-56-05PM.log
The id for this session is YUCM
Please press ENTER button to accept system generated default clone path.
Please enter clone path [/u01/app/oracle/product/19.0.0/dbhome_2 ] :

Please enter clone path [/u01/app/19.0.0/grid_2 ] :

Executing OPatch prereq operations to verify patch applicability on home /u01/app/19.0.0/grid
Patch applicability verified successfully on home /u01/app/19.0.0/grid

Executing OPatch prereq operations to verify patch applicability on home /u01/app/oracle/product/19.0.0/dbhome_1
Patch applicability verified successfully on home /u01/app/oracle/product/19.0.0/dbhome_1

Executing patch validation checks on home /u01/app/19.0.0/grid
Patch validation checks successfully completed on home /u01/app/19.0.0/grid

Executing patch validation checks on home /u01/app/oracle/product/19.0.0/dbhome_1
Patch validation checks successfully completed on home /u01/app/oracle/product/19.0.0/dbhome_1

Verifying SQL patch applicability on home /u01/app/oracle/product/19.0.0/dbhome_1
SQL patch applicability verified successfully on home /u01/app/oracle/product/19.0.0/dbhome_1

Copying the files from the existing oracle home /u01/app/oracle/product/19.0.0/dbhome_1 to a new location. Please wait...

Copying the files from the existing oracle home /u01/app/19.0.0/grid to a new location. Please wait...
Clone of oracle home /u01/app/19.0.0/grid is /u01/app/19.0.0/grid_2 on host ol7asmdb01
Copying the files from the existing oracle home /u01/app/19.0.0/grid to a new location is successful.

Clone of oracle home /u01/app/oracle/product/19.0.0/dbhome_1 is /u01/app/oracle/product/19.0.0/dbhome_2 on host ol7asmdb01
Copying the files from the existing oracle home /u01/app/oracle/product/19.0.0/dbhome_1 to a new location is successful.

Unlocking CRS clone home for home /u01/app/19.0.0/grid.
Prepatch operation log file location: /u01/app/oracle/crsdata/ol7asmdb01/crsconfig/hapatch_2024-06-10_04-14-55PM.log
Unlocked CRS clone home successfully for home /u01/app/19.0.0/grid.

Creating clone for oracle home /u01/app/19.0.0/grid.
Clone operation successful for oracle home /u01/app/19.0.0/grid.

Creating clone for oracle home /u01/app/oracle/product/19.0.0/dbhome_1.
Clone operation successful for oracle home /u01/app/oracle/product/19.0.0/dbhome_1.

Performing post clone operation for oracle home /u01/app/19.0.0/grid.
Performing post clone operation was successful for oracle home /u01/app/19.0.0/grid.

Performing post clone operation for oracle home /u01/app/oracle/product/19.0.0/dbhome_1.
Performing post clone operation was successful for oracle home /u01/app/oracle/product/19.0.0/dbhome_1.

Start applying binary patch on home /u01/app/19.0.0/grid_2
Binary patch applied successfully on home /u01/app/19.0.0/grid_2

Start applying binary patch on home /u01/app/oracle/product/19.0.0/dbhome_2
Binary patch applied successfully on home /u01/app/oracle/product/19.0.0/dbhome_2

Running rootadd_rdbms.sh on home /u01/app/19.0.0/grid_2
Successfully executed rootadd_rdbms.sh on home /u01/app/19.0.0/grid_2

Running rootadd_rdbms.sh on home /u01/app/oracle/product/19.0.0/dbhome_2
Successfully executed rootadd_rdbms.sh on home /u01/app/oracle/product/19.0.0/dbhome_2

Update nodelist in the inventory for oracle home /u01/app/19.0.0/grid_2.
Update nodelist in the inventory is completed for oracle home /u01/app/19.0.0/grid_2.

Bringing down database service on home /u01/app/oracle/product/19.0.0/dbhome_2
Database service successfully brought down on home /u01/app/oracle/product/19.0.0/dbhome_2

Performing postpatch operations on CRS - starting CRS service on home /u01/app/19.0.0/grid_2
Postpatch operation log file location: /u01/app/oracle/crsdata/ol7asmdb01/crsconfig/hapatch_2024-06-10_05-10-38PM.log
CRS service started successfully on home /u01/app/19.0.0/grid_2

Preparing home /u01/app/oracle/product/19.0.0/dbhome_2 after database service restarted
No step execution required.........

Confirm that all resources have been started from home /u01/app/19.0.0/grid_2.
All resources have been started successfully from home /u01/app/19.0.0/grid_2.

Modifying  the database(s) to use new location home /u01/app/oracle/product/19.0.0/dbhome_2.
All database(s) modified successfully to run from new location home /u01/app/oracle/product/19.0.0/dbhome_2.

Trying to apply SQL patch on home /u01/app/oracle/product/19.0.0/dbhome_2
"[/bin/sh -c 'cd /u01/app/oracle/product/19.0.0/dbhome_2;ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_2 ORACLE_SID=crac1d /u01/app/oracle/product/19.0.0/dbhome_2/OPatch/datapatch -verbose']" command failed with errors. Please refer to logs for more details. SQL changes, if any, can be applied by manually retrying the same command.

SQL patch applied successfully on home /u01/app/oracle/product/19.0.0/dbhome_2

OPatchAuto successful.

--------------------------------Summary--------------------------------

Patching is completed successfully. Please find the summary as follows:

Host:ol7asmdb01
SIHA Home:/u01/app/19.0.0/grid
Version:19.0.0.0.0
Summary:

==Following patches were SUCCESSFULLY applied:

Patch: /u01/patch/36233126/36233263
Log: /u01/app/19.0.0/grid_2/cfgtoollogs/opatchauto/core/opatch/opatch2024-06-10_16-20-38PM_1.log

Patch: /u01/patch/36233126/36233343
Log: /u01/app/19.0.0/grid_2/cfgtoollogs/opatchauto/core/opatch/opatch2024-06-10_16-20-38PM_1.log

Patch: /u01/patch/36233126/36240578
Log: /u01/app/19.0.0/grid_2/cfgtoollogs/opatchauto/core/opatch/opatch2024-06-10_16-20-38PM_1.log

Patch: /u01/patch/36233126/36383196
Log: /u01/app/19.0.0/grid_2/cfgtoollogs/opatchauto/core/opatch/opatch2024-06-10_16-20-38PM_1.log

Patch: /u01/patch/36233126/36460248
Log: /u01/app/19.0.0/grid_2/cfgtoollogs/opatchauto/core/opatch/opatch2024-06-10_16-20-38PM_1.log

Host:ol7asmdb01
SIDB Home:/u01/app/oracle/product/19.0.0/dbhome_1
Version:19.0.0.0.0
Summary:

==Following patches were SKIPPED:

Patch: /u01/patch/36233126/36233343
Reason: This patch is not applicable to this specified target type - "oracle_database"

Patch: /u01/patch/36233126/36383196
Reason: This patch is not applicable to this specified target type - "oracle_database"

Patch: /u01/patch/36233126/36460248
Reason: This patch is not applicable to this specified target type - "oracle_database"

==Following patches were SUCCESSFULLY applied:

Patch: /u01/patch/36233126/36233263
Log: /u01/app/oracle/product/19.0.0/dbhome_2/cfgtoollogs/opatchauto/core/opatch/opatch2024-06-10_16-44-45PM_1.log

Patch: /u01/patch/36233126/36240578
Log: /u01/app/oracle/product/19.0.0/dbhome_2/cfgtoollogs/opatchauto/core/opatch/opatch2024-06-10_16-44-45PM_1.log

Patching session reported following warning(s):
_________________________________________________

[Note]: Please verify the database is running from the desired Oracle home, if not then manually execute
 $ORACLE_HOME/bin/srvctl modify database command to fix the problem

Out of place patching clone home(s) summary
____________________________________________
Host : ol7asmdb01
Actual Home : /u01/app/19.0.0/grid
Version:19.0.0.0.0
Clone Home Path : /u01/app/19.0.0/grid_2

Host : ol7asmdb01
Actual Home : /u01/app/oracle/product/19.0.0/dbhome_1
Version:19.0.0.0.0
Clone Home Path : /u01/app/oracle/product/19.0.0/dbhome_2

OPatchauto session completed at Mon Jun 10 19:31:21 2024
Time taken to complete the session 215 minutes, 40 seconds
```
