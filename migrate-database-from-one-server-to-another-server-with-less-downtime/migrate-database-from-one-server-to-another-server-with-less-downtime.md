---
title: "Migrate database from one server to another server with less downtime."
author: "Sunke Nithin"
date: 2022-08-03
categories: ["RMAN"]
tags: []
source: https://nithinsunke.wordpress.com/2022/08/03/migrate-database-from-one-server-to-another-server-with-less-downtime/
---

# Migrate database from one server to another server with less downtime.

1.  take the backup of database

``` wp-block-code
rman> backup database plus archivelog;
```

2\) copy the backup to the new server

``` wp-block-code
scp -pr * oracle@orstsbdvl01:/backup/FTEX
```

3\) create or copy the pfile from the source server

4\) create the directory structure in pfile

5\) start the database in nomount

``` wp-block-code
SQL> startup nomount
ORACLE instance started.

Total System Global Area 1870647296 bytes
Fixed Size                  2254304 bytes
Variable Size             520096288 bytes
Database Buffers         1342177280 bytes
Redo Buffers                6119424 bytes
SQL> exit
```

5\) perform restore and recover through rman

``` wp-block-code
     run {
     allocate channel d1 type disk;
     # set until time "to_date('2002-01-29:15:16:53','yyyy-mm-dd:hh24:mi:ss')";
     restore controlfile from '/backup/FTEX/full/c-2959031005-20220803-00';
     sql 'alter database mount';
     restore database;

     recover database;
}
```

6\) scp the archive log files from source to target server

``` wp-block-code
scp -pr * oracle@orstsbdvl01:/h01/oracle/fast_recovery_area/FTEX/archivelog/2022_08_03
```

7\) recover the target database.

``` wp-block-code
rman> recover database;
```

8\) repeat the above 6,7 steps hourly/daily

9\) on cutover day perform a manual log switch and bring down the database copy the archive logfiles to target server

``` wp-block-code
sql> shut immediate

$ scp -pr * oracle@orstsbdvl01:/h01/oracle/fast_recovery_area/FTEX/archivelog/2022_08_03
```

10\) recover database and open the database with resetlogs

``` wp-block-code
rman> recover database;

sql> alter database open resetlogs;
```

11\) now point the application to the new server and perform the application validations.
