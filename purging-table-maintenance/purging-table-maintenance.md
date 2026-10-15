---
title: "Purging table Maintenance"
author: "Sunke Nithin"
date: 2024-06-04
categories: ["ORACLE"]
tags: ["DATABASE", "performance", "SQL"]
source: https://nithinsunke.wordpress.com/2024/06/04/purging-table-maintenance/
---

# Purging table Maintenance

## Scenario:

Customer has a table cust_data with around 20 million records.  
as per the company policy then want to do the old data archival which is no longer required.

The table consists of data from March 2009 to February 2018. As per policy they decided to archive data older then 5 years (older then February 2013).

cust_data is the non-partitioned table and want to perform this operation with minimum down time.

### Steps:

1.  Convert the table to partitioned table
2.  Drop the individual partitions to archive the data.

### **Create a sample table for demo.**

``` wp-block-code
CREATE TABLE cust_data ( customer_id NUMBER, customer_name VARCHAR2(100), phone_number VARCHAR2(20), cust_date DATE );
```

**Now populate the demo data with procedure**

``` wp-block-code
CREATE OR REPLACE PROCEDURE insert_cust_data AS
    v_batch_size NUMBER := 1000; -- Adjust batch size as needed
    v_start_date DATE := TO_DATE('01-MAR-2009', 'DD-MON-YYYY');
    v_end_date DATE := TO_DATE('28-FEB-2018', 'DD-MON-YYYY');
BEGIN
    FOR i IN 1..CEIL(100000000 / v_batch_size) LOOP
        FOR j IN 1..v_batch_size LOOP
            INSERT INTO cust_data (customer_id, customer_name, phone_number, cust_date)
            VALUES (
                (i - 1) * v_batch_size + j,
                'Customer_' || ((i - 1) * v_batch_size + j),
                '123-456-' || LPAD(TRUNC(DBMS_RANDOM.VALUE(0, 9999)), 4, '0'),
                v_start_date + TRUNC(DBMS_RANDOM.VALUE(0, v_end_date - v_start_date))
            );
        END LOOP;
        COMMIT; -- Commit after each batch
    END LOOP;
END;
/
```

**Now execute the procedure**

``` wp-block-code
SQL> EXECUTE insert_cust_data;
PL/SQL procedure successfully completed.
```

**Create index**

``` wp-block-code
CREATE UNIQUE INDEX "NSUNKE"."CUST_INDX" ON "NSUNKE"."CUST_DATA" ("CUSTOMER_ID") PARALLEL ;
```

#### **Create a primary key**

In order to perform table re-org atleast one column should have a primary key

``` wp-block-code
alter table "NSUNKE"."CUST_DATA" add constraint CUST_ID_PK primary key("CUSTOMER_ID")
```

**Create a partitioned table**

to convert the non-partition table to partition table we will create same table structure with month wise partition.

``` wp-block-code
CREATE TABLE cust_data_part (
CREATE TABLE cust_data_part (
    customer_id NUMBER,
    customer_name VARCHAR2(100),
    phone_number VARCHAR2(20),
    cust_date DATE
)
PARTITION BY RANGE (cust_date)
(
    PARTITION part_mar_2009 VALUES LESS THAN (TO_DATE('01-MAR-2009', 'DD-MON-YYYY')),
    PARTITION part_apr_2009 VALUES LESS THAN (TO_DATE('01-APR-2009', 'DD-MON-YYYY'))
```

Add the remaining partitions using below script

``` wp-block-code
DECLARE
    v_start_date DATE := TO_DATE('01-MAY-2009', 'DD-MON-YYYY');
    v_end_date DATE := TO_DATE('28-FEB-2018', 'DD-MON-YYYY');
    v_partition_name VARCHAR2(100);
    v_partition_bound DATE := v_start_date;
BEGIN
    WHILE v_partition_bound < v_end_date LOOP
        -- Generate partition name based on month and year
        v_partition_name := 'PART_' || TO_CHAR(v_partition_bound, 'MON_YYYY');
        
        -- Create partition with range for the current month
        EXECUTE IMMEDIATE '
            ALTER TABLE cust_data_part
            ADD PARTITION ' || v_partition_name || ' 
            VALUES LESS THAN (TO_DATE(''' || TO_CHAR(ADD_MONTHS(v_partition_bound, 1), 'DD-MON-YYYY') || ''', ''DD-MON-YYYY''))';
        
        -- Move to the next month
        v_partition_bound := ADD_MONTHS(v_partition_bound, 1);
    END LOOP;
END;
/
```

Create final max value partition

``` wp-block-code
ALTER TABLE cust_data_part ADD PARTITION "PART_MAR_2018"  VALUES LESS THAN (MAXVALUE);
```

**List the rows count month wise**

``` wp-block-code
SELECT 
    TO_CHAR(cust_date, 'YYYY-MM') AS month,
    COUNT(*) AS total_records
FROM 
    cust_data
GROUP BY 
    TO_CHAR(cust_date, 'YYYY-MM')
ORDER BY 
    month;
```

Always better to take a backup before doing any maintenance.

``` wp-block-code
[oracle@ol7_19c_vagrant admin]$ expdp directory=dpump dumpfile=cust_data_%U.dmp logfile=cust_data.log tables=NSUNKE.CUST_DATA parallel=4

Export: Release 19.0.0.0.0 - Production on Mon Jun 3 13:25:22 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Username: / as sysdba
Password:

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Starting "SYS"."SYS_EXPORT_TABLE_01":  /******** AS SYSDBA directory=dpump dumpfile=cust_data_%U.dmp logfile=cust_data.log tables=NSUNKE.CUST_DATA parallel=4
Processing object type TABLE_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Processing object type TABLE_EXPORT/TABLE/TABLE
Processing object type TABLE_EXPORT/TABLE/STATISTICS/MARKER
. . exported "NSUNKE"."CUST_DATA"                        1.329 GB 29380000 rows
Master table "SYS"."SYS_EXPORT_TABLE_01" successfully loaded/unloaded
******************************************************************************
Dump file set for SYS.SYS_EXPORT_TABLE_01 is:
  /nfsshare/oracle/dpump/cust_data_01.dmp
  /nfsshare/oracle/dpump/cust_data_02.dmp
  /nfsshare/oracle/dpump/cust_data_03.dmp
Job "SYS"."SYS_EXPORT_TABLE_01" successfully completed at Mon Jun 3 13:26:29 2024 elapsed 0 00:00:46
```

## Online Table Redefinition Including Dependents (COPY_TABLE_DEPENDENTS)

``` wp-block-code
SQL>  EXEC DBMS_REDEFINITION.can_redef_table('NSUNKE', 'CUST_DATA');

PL/SQL procedure successfully completed.
```

it is good if we have more resources available on server increase parallelism to desired level for large tables. If database is on cloud you can scale CPU's on demand and scale out once the maintenance is completed.

``` wp-block-code
ALTER SESSION FORCE PARALLEL DML PARALLEL 4;
ALTER SESSION FORCE PARALLEL QUERY PARALLEL 4;
```

``` wp-block-code
SET SERVEROUTPUT ON
DECLARE
  v_redef_errors VARCHAR2(4000);
   l_num_errors PLS_INTEGER;
BEGIN
  DBMS_REDEFINITION.start_redef_table('NSUNKE', 'CUST_DATA', 'CUST_DATA_PART');
  DBMS_REDEFINITION.sync_interim_table('NSUNKE', 'CUST_DATA', 'CUST_DATA_PART'); 
  DBMS_REDEFINITION.copy_table_dependents(
    uname               => 'NSUNKE',
    orig_table          => 'CUST_DATA',
    int_table           => 'CUST_DATA_PART',
    copy_indexes        => 1,            
    copy_triggers       => TRUE,          
    copy_constraints    => TRUE,          
    copy_privileges     => TRUE,          
    ignore_errors       => FALSE,         
    num_errors          => l_num_errors,
    copy_statistics     => FALSE,         
    copy_mvlog          => FALSE); 
  DBMS_REDEFINITION.finish_redef_table('NSUNKE', 'CUST_DATA', 'CUST_DATA_PART'); 
EXCEPTION
  WHEN OTHERS THEN
    -- Handle errors
    v_redef_errors := SQLERRM;
    DBMS_OUTPUT.PUT_LINE('Error: ' || v_redef_errors);
    -- Rollback any changes if necessary
    DBMS_REDEFINITION.ABORT_REDEF_TABLE(
      uname => 'NSUNKE',
      orig_table => 'CUST_DATA',
      int_table => 'CUST_DATA_PART'
    );      
END;
/
```

before dropping the intermediate table compare the row count of both tables

``` wp-block-code
SQL> select count(*) from nsunke.CUST_DATA_PART;

  COUNT(*)
----------
  29380000

SQL> select count(*) from nsunke.CUST_DATA;

  COUNT(*)
----------
  29380000

SQL> DROP TABLE nsunke.CUST_DATA_PART;
Table dropped.
```

Gather the table stats

``` wp-block-code
exec dbms_stats.gather_table_stats(ownname=>'NSUNKE',tabname=>'CUST_DATA',estimate_percent=>DBMS_STATS.AUTO_SAMPLE_SIZE,cascade=>TRUE,degree =>4);
```

**CHECK FOR any unable indexes**

**I**f there are any unusable index  rebuild index

``` wp-block-code
SQL> alter index nsunke.CUST_INDX rebuild online;

SQL> select index_name , status , table_name from dba_indexes where status ='UNUSABLE';

no rows selected
```
