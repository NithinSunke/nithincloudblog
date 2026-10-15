---
title: "RMAN Useful Commands"
author: "Sunke Nithin"
date: 2025-04-16
categories: ["ORACLE"]
tags: ["DATABASE"]
source: https://nithinsunke.wordpress.com/2025/04/16/rman-useful-commands/
---

# RMAN Useful Commands

Oracle **Recovery Manager (RMAN)** is the recommended utility for efficiently backing up, restoring, and recovering Oracle databases. It offers powerful features to manage database backups, validate data integrity, and perform disaster recovery.

In this post, let’s compile a list of commonly used and practical **RMAN commands** for Oracle DBAs.

``` wp-block-code
This will list all RMAN configuration parameters.
RMAN> show all;
This will show the complete list of all backups from repository for database, control file, archive logs and spfile.
RMAN> list backup;
This will display complete list of all backups from repository for database, control file, archive logs and spfile, except the current date i.e. sysdate-1.
RMAN> list backup completed before 'sysdate-1';
This will display complete list of all backups from repository for database only, not for control file, spfile, and archive logs. This will not display current date backups i.e. sysdate-1.
RMAN> list backup of database completed before 'sysdate-1';
This will show you all backups of archive logs from repository.
RMAN> list backup of archivelog all;
Use below command to list backup of specific archive log sequence;
RMAN> list backup of archivelog sequence 192;
Use below command to list backup of archive logs until specific sequence number;
RMAN> list backup of archivelog until sequence 192;
Use below command to list backup of archive logs from specific sequence number.
RMAN> list backup of archivelog from sequence 186;
Use below command to list backup of archive logs between specific sequence numbers.
RMAN> list backup of archivelog sequence between 186 and 192;
Use below command to list backup of specific TAG.
RMAN> list backup TAG 'TAG20230225T200006';

This will show the backup summary in short in tabular format.
RMAN> list backup summary;
Use below command to list backup summary of specific TAG in tabular format.
RMAN> list backup summary TAG 'TAG20230225T200006';
Use below command to list backup summary of all archive logs in tabular format.
RMAN> list backup of archivelog all summary;

Use below command to list backup summary of specific archive log sequence number in tabular format.
RMAN>  list backup of archivelog until sequence 192 all summary;
OR
RMAN>  list backup of archivelog until sequence 192 summary;

Use below command to list backup summary of archive log between two sequence numbers in tabular format.
RMAN> list backup of archivelog sequence between 186 and 192 all summary;
OR 
RMAN> list backup of archivelog sequence between 186 and 192 summary;

This will display short summary of all backups from repository for database, control file, archive logs and spfile, except the current date i.e. sysdate-1.
RMAN> list backup summary completed before 'sysdate-1';
This will display short summary of all backups from repository for database only, except the current date i.e. sysdate-1.
RMAN>  list backup of database summary completed before 'sysdate-1';
This will list specific backuppiece.
RMAN> list backuppiece 'D:\RUPESH\SETUPS\WINDOWS.X64_193000_DB_HOME\DATABASE\C-2972374659-20230301-00';
This will list backup of spfile only.
RMAN> list backup of spfile;
This will list backup summary of spfile only.
RMAN> list backup of spfile summary;
This will list backup of controlfile only.
RMAN> list backup of controlfile summary;
This will list backup summary of controlfile only.
RMAN> list backup of controlfile summary;
This will list backup of specific datafile only.
RMAN> list backup of datafile 1 summary;
This will list backup summary of specific datafile only.
RMAN> list backup of datafile 1 summary;
This will list backup of multiple datafiles.
RMAN> list backup of datafile 1,2 summary;
This will list backup summary of multiple datafiles.
RMAN> list backup of datafile 1,2 summary;

This will list backup of specific tablespace.
RMAN> list backup of tablespace system;

This will list backup summary of specific tablespace.
RMAN> list backup of tablespace system summary;

This will list backup of multiple tablespaces.
RMAN> list backup of tablespace system,sysaux summary;
This will list backup summary of multiple tablespaces.
RMAN> list backup of tablespace system,sysaux summary;
This will list backupsets in repository.
RMAN> list backupset;
This will list summary of backupsets in repository.
RMAN> list backupset summary;
This will list all incarnations in the repository.
RMAN> list incarnation;
This will list backup of specific TAG by using Like operator.
RMAN> list backup tag like '%TAG2023022%';
This will list all db_unique_name in the repository.
RMAN> list db_unique_name all;
This will list all restore points from the repository.
RMAN> list restore point all;
This will crosscheck all backups (datafile + controlfile + spfile ) , except archive logs.
RMAN> crosscheck backup;

This will crosscheck all archive logs.
RMAN> crosscheck archivelog all;
This will crosscheck specific archive log sequence.
RMAN> crosscheck archivelog sequence 180;
This will crosscheck archive log between two sequences.
RMAN> crosscheck archivelog sequence between 180 and 196;
This will crosscheck archive logs until specific sequence.
RMAN> crosscheck archivelog until sequence 196;
This will crosscheck archive log sequences from sequence to until sequence.
RMAN> crosscheck archivelog from sequence 180 until sequence 196;

This will crosscheck specific backuppiece.
RMAN> crosscheck backuppiece 'D:\RUPESH\SETUPS\WINDOWS.X64_193000_DB_HOME\DATABASE\C-2972374659-20230225-04';
This will crosscheck all backupsets in repository.
RMAN> crosscheck backupset ;
This will delete all archive logs from repository as well as from physical location. This will ask input once whether you want to really delete.
RMAN> delete archivelog all;
This will delete all archive logs from repository as well as from physical location. This will not ask input once whether you want to really delete, instead it will directly delete.
RMAN> delete noprompt archivelog all;
This will delete specific archive log sequence from repository as well as from physical location.
RMAN> delete archivelog sequence 180;
OR
RMAN> delete noprompt archivelog sequence 180;

This will delete specific archive log sequence from repository as well as from physical location.
RMAN> delete archivelog until sequence 190;
OR
RMAN> delete noprompt archivelog until sequence 190;
This will delete archive logs from sequence to until sequence from repository as well as from physical location.
RMAN> delete archivelog from sequence 180 until sequence 190;
OR
RMAN> delete noprompt archivelog from sequence 180 until sequence 190;
This will report names and locations of all Permanent and temporary tablespaces along with data files, identifiers, and size.
RMAN> report schema;
This will catalog the backups located in specific directory.
RMAN> catalog start with '/u01/backup/';

This will catalog the backups located in specific directory.
RMAN> catalog start with '/u01/backup/';

This will catalog complete Flash Recovery Area.
RMAN> catalog db_recovery_file_dest ;
OR
RMAN> catalog recovery area;
```
