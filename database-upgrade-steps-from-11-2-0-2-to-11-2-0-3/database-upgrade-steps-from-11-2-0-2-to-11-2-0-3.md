---
title: "Database Upgrade steps from 11.2.0.2 to 11.2.0.3"
author: "Sunke Nithin"
date: 2019-06-13
categories: ["Database"]
tags: ["Oracle", "Upgrade"]
---

# Database Upgrade steps from 11.2.0.2 to 11.2.0.3

**Important Notes:**  
Metalink \[ID 730365.1\] Includes all patchset downloads + How to upgrade from any Oracle DB version to another one.  
Metalink \[ID 1276368.1\] Out-of-place manual upgrade from previous 11.2.0.N version to the latest 11.2.0.N patchset.

**Before Starting The Upgrade Process:**  
Take the RMAN backup of database.

Backup & Truncate Audit Table SYS.AUD\$:  
Truncating SYS.AUD\$ table will speed up the upgrade process:

Backing up and compressing audit data in SYS.AUD_BKP table:  

    SQL> create table SYS.AUD_BKP COMPRESS as select * from SYS.AUD$;

    SQL>Select count(*) from sys.aud$;

    SQL> Truncate table SYS.AUD$;

  
**Purge DBA_RECYCLEBIN:**  
Purging DBA_RECYCLEBIN will speed up the upgrade process:  

    SQL> PURGE DBA_RECYCLEBIN;

  
**Save a backup of all DB Parameters (Visible & hidden):**  
This will help you in troubleshooting issues coming after the upgrade (bugs, performance, or whatever the problem is).

The following statement will make it easy for you:  

    SQL> Spool All_parameters.txt

    set linesize 170

    col Parameter for a50

    col "Session Value" for a40

    col "Instance Value" for a40

    SELECT a.ksppinm "Parameter", b.ksppstvl "Session Value", c.ksppstvl "Instance Value" FROM x$ksppi a, x$ksppcv b, x$ksppsv c WHERE a.indx = b.indx AND a.indx = c.indx order by 1;

    Spool off

  
I also strongly recommend to reset the hidden parameter to its default values before starting the upgrade, you don't want to get a weird problems during the upgrade and you don't know from where they came up.

**PRE-UPGRADE STEPS:**

**Step 1: Software Installation**

Install 11.2.0.3 RDBMS Software into a new ORACLE_HOME.  
=\>You only need to install examples cd if your DB is using Oracle Text Themes Apply the latest PSU patch on the 11.2.0.3 ORACLE_HOME before upgrading the database.  
PSU patch post steps which runs on the database like executing script  

    $ORACLE_HOME/rdbms/admin/catbundle.sql

  
are not required in my case as per 11.2.0.3.4 PSU read-me.  
Note that every PSU patch is unique, not all PSU patches exempt running catcpu.sql after the database upgrade, so you have to read the readme file first.

**Pre-upgrade information:**

Download latest version of utlu112i_5.sql script and execute it:  
Note ID 884522.1  
Note: This script already exist under the new Oracle Home \$ORACLE_HOME/rdbms/admin, but usually the one in Metalink is updated with the most recent upgrade checks.  

    SQL> SPOOL upgrade_info.log

    SQL> @/u02/app/oracle/product/11.2.0.3/db /rdbms/admin/utlu112i.sql

    SQL> SPOOL OFF

  
**Step 2: Dictionary Check**  
Verify the validity of data dictionary objects by running dbupgdiag.sql script (Download it from NOTE ID 556610.1)

If the **dbupgdiag.sql** script reports any invalid objects, run utlrp.sql (multiple times) to validate the invalid objects in the database, until there is no change in the number of invalid objects:  

    SQL> @//u02/app/oracle/product/11.2.0.3/db /rdbms/admin/dbupgdiag.sql

    SQL> @?/u02/app/oracle/product/11.2.0.3/db /rdbms/admin/utlrp.sql

  
**Gather Dictionary Statistics:**  
Helps in speeding up upgrade process.  

    SQL> EXECUTE dbms_stats.gather_dictionary_stats;

  
**Step 3: TIMEZONE Version**

In case you have TIMESTAMP WITH TIME ZONE data type in your DB, you need to Upgrade the Time Zone version to version 14.

The possibilities are:  
\>If current version \< 14 ,You need to upgrade to version 14 after you finish the upgrade to 11.2.0.3.  
\>If current version = 14 ,No need to upgrade, Skip the whole Step.  
\>If current version \> 14 ,You must upgrade your Time Zone version before upgrading to 11.2.0.3 or your data stored in TIMESTAMP WITH TIME ZONE datatype can become corrupted during the upgrade.

Check your current Time Zone version:  

    SQL> SELECT version FROM v$timezone_file;

    VERSION

    ----------

    4

  
In my case the version is older, so I can do this step later after finalizing the upgrade.  

    Check National Characterset is UTF8 or AL16UTF16:

    SQL> select value from NLS_DATABASE_PARAMETERS where parameter = 'NLS_NCHAR_CHARACTERSET';

  
Step 4: Disable Cluster option  
Set the parameter cluster_database to FALSE for RAC database.

**Step 5: Configure log locations**

Change SPFILE parameters to point to the new ORACLE HOME:  

    # mkdir -p /u02/app/oracle/product/11.2.0.3/db /diagnostics/orcl

    SQL> ALTER SYSTEM SET diagnostic_dest = '/u02/app/oracle/product/11.2.0.3/db /diagnostics/orcl';

    SQL> ALTER SYSTEM SET audit_file_dest = '/u02/app/oracle/product/11.2.0.3/db /rdbms/audit' SCOPE=SPFILE;

  
**Step 6: New Environment Variables**

Make sure the following Linux environment variables are pointing to the new 11.2.0.3 ORACLE_HOME:  
(ORACLE_BASE, ORACLE_HOME, PATH, NLS_10 and LIBRARY_PATH).

**Step 7: Modify \| Move configuration files**

\>Make sure that entries inside /etc/oratab file are pointing to the new 11.2.0.3 ORACLE_HOME, hash the original entries pointing to 11.2.0.2 ORACLE_HOME:  
e.g.  
\#dbprod: /u01/app/oracle/product/11.2.0.2/db:Y  
dbprod: /u02/app/oracle/product/11.2.0.3/db:Y

Copy SPFILE & Password File to the new ORACLE_HOME:  

    # cd //u01/app/oracle/product/11.2.0.2/db /dbs

    # cp spfile* orapw* /u02/app/oracle/product/11.2.0.3/db/dbs/

  
Copy network configuration files to the new 11.2.0.3 \$TNS_ADMIN directory:  

    # cd $ORACLE_HOME/network/admin

    # cp tnsnames.ora listener.ora sqlnet.ora /u02/app/oracle/product/11.2.0.3/db/network/admin/

  
Copy DB Control EM directories to the new 11.2.0.3 ORACLE_HOME:  

    # $ORACLE_HOME/

    # $ORACLE_HOME/oc4j/j2ee/OC4J_DBConsole_

    # $ORACLE_HOME/owb/bin/admin

  
Copy SQLPLUS settings file:  

    # cp /u01/app/oracle/product/11.2.0.2/db/sqlplus/admin/glogin.sql /u02/app/oracle/product/11.2.0.3/db/sqlplus/admin/

  
**Step 8: Disable Vault \| Adjust parameters for JVM**

Disable Database Vault if enabled.

IF JVM installed, java_pool_size and shared_pool_size must be set to at least 250MB prior to the upgrade.  

    Latest checks:

    SELECT * FROM v$recover_file; no rows selected

    SELECT * FROM v$backup WHERE status != 'NOT ACTIVE'; 

    SELECT * FROM dba_2pc_pending; --outstanding distributed transactions;

    SELECT name FROM sys.user$ WHERE ext_username IS NOT NULL AND password = 'GLOBAL';

  
Disable all batch and cron jobs:

Disable all crontab scripts:  

    # crontab /root/crontab_root

    # crontab /dev/null

    # crontab -l

    # crontab -l > /home/oracle/oracle_crontab

    # crontab /dev/null

    # crontab -l

  
Disable DB jobs:  

    SQL> EXEC dbms_scheduler.set_scheduler_attribute('SCHEDULER_DISABLED','TRUE');SQL> alter system set job_queue_processes=0 scope=both;

  
Step 9: Set the Database in the Noarchivelog mode.

Stop the listener: lsnrctl stop  
Stop DBCONSOLE: emctl stop dbconsole

Putting the database in the noarchivelog shrink the upgrade time. .  

    $ sqlplus "/as sysdba"

    SQL> shutdown immediate;

    SQL> startup mount

    SQL> alter database noarchivelog;

    SQL> archive log stop;

    SQL> shutdown immediate;

  
YOU MUST TAKE A COLD BACKUP AT THIS STAGE FOR FALLBACK PLAN.

**UPGRADE STEPS:**

Note: The database should be opened from the old ORACLE_HOME before running DBUA.  
Note: If Oracle clusterware installed, it needs to be UP and running before starting DBUA.  
DBUA Way: (GUI)  
===========  
Ensure that dbua is running from the new ORACLE_HOME:  
\# which dbua

Run the DBUA by Oracle User:  
\# dbua  

    =>First screen: introduction .

    =>2nd screen: Select the database name.

    =>3rd screen: Check the following:

    -Recompile invalid objects at the end of upgrade.

    -Turn off Archiving for the duration of upgrade.

    -Upgrade Timezone version and TIMESTAMP WITH TIMEZONE data.

    =>4th screen: keep the default.

    =>5th screen: keep the default.

    =>6th screen: keep the default.

    =>7th screen: Put password for DBSNMP,SYSMAN.

    =>8th screen: Select the DB listener.

    =>9th screen: Finish.

  
Notes: Continue to Post upgrade steps but skip these two steps (Upgrade the Timezone version & Configure EM) as they are automatically configured by DBCA.

The following are Optional STEPS:

**STEP 20: Rebuild Unusable Indexes & Gather Statistics**

Rebuild Unusable indexes:  

    SQL> select 'ALTER INDEX '||OWNER||'.'||INDEX_NAME||' REBUILD ONLINE;'from dba_indexes where status ='UNUSABLE';

  
Gather FIXED OBJECTS stats: (Do it during peak hours not within the downtime)

Fixed objects are the x\$ tables (been loaded in SGA during startup) on which V\$ views are built (V\$SQL etc.).  
Fixed Object statistics are not being gathered automatically nor within gather DB stats procedure.  
If the statistics are not gathered on fixed objects, the Optimizer will use predefined default values for the statistics. These defaults may lead to inaccurate execution plans.  
Statistics on fixed objects are not being gathered automatically nor within gathering DB stats.  
Note:  
-It's recommended to Gather the fixed object stats during peak hours (system is busy) or after the peak hours but the sessions are still connected (even if they idle), to gurantee that the fixed object tables been populated and the statistics well represent the DB activity. also note that performance degradation may be experienced while the statistics are gathering.  
-Having no statistics is better than having a non representative statistics.

Gather the fixed objects stats:  

    SQL> exec dbms_stats.gather_fixed_objects_stats;

  
Gather DICTIONARY stats:  
Dictionary stats are gathered on dictionary tables owned by SYS and resides in the system tablespace.  

    SQL> Exec DBMS_STATS.GATHER_DICTIONARY_STATS ();

  
Gather database statistics:  

    SQL> EXEC DBMS_STATS.GATHER_DATABASE_STATS(ESTIMATE_PERCENT=>DBMS_STATS.AUTO_SAMPLE_SIZE,degree => 8);

  
In case you need to gather DB Stats + Histograms on all skewed columns:  

    SQL> EXEC DBMS_STATS.GATHER_DATABASE_STATS(ESTIMATE_PERCENT=>100,METHOD_OPT=>'FOR ALL COLUMNS SIZE SKEWONLY',cascade => TRUE,degree => 8);

  
ESTIMATE_PERCENT=\>DBMS_STATS.AUTO_SAMPLE_SIZE =\> Let Oracle estimate skewed values always gives excellent results.  
Removed "METHOD_OPT=\>'FOR ALL COLUMNS SIZE SKEWONLY'" =\> As histograms is not recommended to gathered on all columns.  
Removed "cascade =\> TRUE" =\> To let Oracle determine whether index statistics to be collected or not.

**Step 21: Check Oracle Recommended Patches:**

Note ID 756671.1 includes the recommended patches for releases starting from 10.2.0.3.  
Note ID 742060.1 represents the most accurate information Oracle can provide for coming releases.  
\*You can check the latest applied patches on the database by running this query:  
SQL\> select \* from DBA_REGISTRY_HISTORY;

Removing OLD ORACLE_HOME: (Optional, you can do it later)

After you feel confident with the new Oracle installation and you will never downgrade to the previous release, remove the ORACLE_HOME:

Detach old ORACLE_HOME:  

    # $OLD_HOME/oui/bin/runInstaller -detachHome -silent -local

  
Confirm old ORACLE_HOME is removed from central inventory:  

    # $OLD_HOME/OPatch/opatch lsinventory -all

  
Remove files in old ORACLE_HOME manually:  
\# rm -rf \$OLD_HOME
