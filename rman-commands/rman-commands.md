---
title: "RMAN commands"
author: "Sunke Nithin"
date: 2019-06-13
categories: ["RMAN"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/13/rman-commands/
---

# RMAN commands

**Allocate channel:**

To manually allocate a channel, which is a connection between RMAN and a database instance. Each connection initiates an Oracle server session on the target or auxiliary instance: this server session performs the work of backing up, restoring, or recovering backup sets and copies. Each channel operates on one backup set at a time(for BACKUP, RESTORE, or RECOVER) or one image copy at a time (for COPY). RMAN automatically releases the channel at the end of the job.  
Example 1:

This command allocated a tape channel for a whole database backup:  

    RMAN> RUN

    {

    ALLOCATE CHANNEL dev1 DEVICE TYPE sbt;

    BACKUP DATABASE;

    }

  
**Example:2**

When creating multiple copies of a backup set, you can specify the SET BACKUP COPIES command. The following example generates a single backup of the database, and then creates four identical backups of datafile 1 to four file systems:  

    RMAN> RUN

    {

    ALLOCATE CHANNEL dev1 DEVICE TYPE DISK MAXPIECESIZE 2M;

    BACKUP DATABASE;

    SET BACKUP COPIES = 4;

    BACKUP DATAFILE 1FORMAT ‘/dsk1/bp/%U’ , ‘/dsk2/sv/%U’ ,/dsk3/bp/%U’ , ‘/dsk3/sv/%U’ ;

    }

  
**1.2 BACKUP:**  

To back up a database, tablespace, datafile (current or copy), control file current or copy), archived log, or backup set. You can back up a target or standby database.

  

When performing abackup, specify the files that you want to backup. RMAN putsthe input file into one or more backup sets, which are RMAN-specific logical structures. The backup set is the smallest unit of a backup. RMAN only recordsbackup sets in the repository that complete successfully. Each backup set contains at least one backup piece, which is a physical file containing the backed up data.

  

FULL – copies all blocks into the backup set, skipping only datafile blocks that have never been used. RMAN makes full backups by default if neither FULL nor INCREMENTAL is specified. The server session does not skip blocks when backing up archived redo logs or control files. A full backup has no effect on subsequent incremental backups, so it is not considered a part of the incremental backup strategy.  
INCREMENTAL LEVEL = integer - Copies only those data blocks that have changed since the last incremental integer backup, where integer is any integer from 1 to 4. for example, in a level 2 backup RMAN backs up all blocks used since the most recent level 2, level 1, or level 0 backup. This type of incremental backup is also called a differential backup to distinguish it from a cumulative backup.  
If you specify INCREMENTAL, then in the backupSpec clause you must set one of the following parameters: DATAFILE, DATAFIELCOPY, TABLESPACE, or DATABASE.RMAN does not support incremental backups of control files, archived redologs, or backup sets.  
BACKUPSET – Backs up either ALL backup sets or backup sets specified by primary_key or completion time. Use this parameter in conjunction with the DEVICE TYPE sbt clause to back up all backups on disk to tape. You cannot back up from tape to tape or from tape to disk: only from disk to disk or disk to tape.  
DATAFILE – Specifies a list of one or more datafiles.  
DATAFILECOPY ‘filename’ -Specifies the filenames of one or more datafile image copies.  
TABLESPACE tablespace_name – specifies the names of one or more tablespaces. RMAN backs up all datafiles that are currently part of the tablespaces.  
DATABASE - specifies datafiles in the database. To include the current control file in the backup set, specify the INCLUDE CURRENT CONTROLFILE clause.  
CURRENT CONTROLFIEL - specifies the current control file.  
CHANNEL channel_id -Specifies the case-sensitive name of a channel to use when creating backup sets. Use any name that is meaningful, for example ch1or dev1.  
CUMULATIVE – Copies the data blocks used since the most recent backup at level n-1 or lower, where in is an integer from 1 to 4. For example, in a cumulative level 2 backup RMAN backs up all bloks used since the most recentlevel 1 or level 0 backup.  
MAXSETSIZE = integer -Specifies a maximum size for a backup set in bytes Set in bytes (default), kilobytes (K), megabytes (M), and gigabytes (G0. Thus, to limit a backup set to 3 MB, specify MAXSETSIZE = 3M.  
FILESPERSET = integer -Specifies a maximum number of imput files in each backup set. If you set FILEPERSET = N, then RMAN never includes more than n files in a backup set.  
NOT BACKED UP – Backs up only those files (of the files specified on the command)that RMAN has not backed up since the specified time. If SINCE TIME is not specified, then only those files that have never been backed up will be backed up.

  
SINCE TIME = ‘data_string’

Specifies the date after which RMAN should back up files that have no backups. The data_string is either a date in the current NLS_DATE_FORMAT, or a SQL date expression such as SYSDATE -1’

DEVICE TYPE - Allocates automatic channels for the specified device type only.  

    BACKUP DEVICE TYPE DISK DATABASE;

  
INCLUDE CURRENT CONTROLFILE - Create a snapshot of the current control file and places it into each backup set produced by this clause.  
DELETE INPUT - deletes the input files upon successful creation of the backup set. Specify this option only when backing up archived logs, datafile copies, or backup sets.  
FORMAT - Specifies a filename to use for the backup piece. If you do not specify the FORMAT parameter, RMAN stores the backup pieces in a port-specific directory (\$ORACLE_HOME/dbs on UNIX).

   

  

  

  

%c

  

Specifies the copy number of the backup piece within a set of duplexed backup pieces.

  

  

  

%d

  

Specifies the name of the database

  

  

  

%D

  

Specifies the current day of the month from the Gregorian calendar in format DD.

  

  

  

%F

  

Combines the DBID, day,  month, year, and sequence  into a unique and repeatable generated name.

  

  

  

%M

  

Specifies the month in the Gregorian calendar in format MM

  

  

  

%n

  

Specifies the name of the database, padded on the right with x characters to a total length of eight Characters.

  

  

  

%P

  

Specifies the piece number within the backup set.

  

  

  

%s

  

Specifies the backup set number.

  

  

  

%t

  

Specifies the backup set time stamp, which is a 4-byte cvalue derived as the number of seconds elapsed since a fixed reference time.

  

  

  

%T

  

Specifies the year, nonth, and day in this format: YYYYMMDD.

  

  

  

%u

  

Specifies an 8 – character name constituted by compressed representations of the backup set number and the time the backup set was created.

  

  

  

%U

  

Specifies convenient shorthand for %u\_%p\_%c  that guarantees uniqueness in generated backup filenames.

  

  

  

%Y

  

Specifies the year in this format:YYYY.

  

  

  

%%

  

Specifies the ‘%’ character.  For example,  %%Y  translates  to the string  %Y.

  

  

  

  
 

**Exampel 1:**

Backing Up a Database: This command backs up the database and then backs up the control file that contains the record of the database backup:  

    BACKUP DATABASE;

    BACKUP CURRENT CONTROLFILE;

  
**Example 2:**

Backing Up tablespaces and Datafiles: This command uses two backupSpec clauses to backup tablespaces and datafiles and lets RMAN perform automatic parallelization of the backup:  

    RUN

    {

    ALLOCATE CHANNEL dev1 DEVICE TYPE DISK FORMAT ‘/fs1/%U’ ;

    ALLOCATE CHANNEL dev2 DEVICE TYPE DISK FORMAT ‘/fs2/%U’ ;

    BACKUP

    (TABLESPACE SYSTEM,sales1,sales2,sales3 FILESPERSET 20)

    (DATAFILE 12, 14, 15);

    }

  
**Example 3:**

Backingt Up Multiple Copies of Archived Logs and Deleting the input:  

    BACKUP ARCHIVELOG LIKE ‘ /arch/dest%’ DELETE ALL INPUT;

  
**Example 4:**

Backing Up Backup Sets to Tape: In this example, you want to keep recent backup sets on disk and older backup sets on tape. You do not want backup sets to exist on disk and tape simultaneously. Hence, you execute this command to back up older backups created more than two weeks ago to tape and then delete the input backup pieces:  

    BACKUP DEVICE TYPE sbt BACKUPSET CREATED BEFORE ‘SYSDATE – 14 ‘ DELETE INPUT;

  
**Example 5:**

Specifying DEVICE TYPE on the BCAKUP Command: This example configures DISK as the default device type, and then backs up archived log to tape:  

    CONFIGURE DEFAULT DEVICE TYPE TO DISK;

    BACKUP DEVICE TYPE sbt ARCHIVELOG ALL;

  
**Example 6:**

Performing a Cumulncremental Backup: This example backs up all blocks changed in the database since the most recent level 0 or level 1 backup:  

    BACKUP INCREMENTAL LEVEL 2 CUMULATIVE SKIP INACCESSIBLE DATABASE;

  
**Example 7:**

Checking for Corruption: This example backs up datafile 3 and specifies that no more than two blocks with corruption should be tolerated:  

    RUN

    {

    SET MAXCORRUPT FOR DATAFILE 3 TO 2;

    BACKUP CHECK LOGICAL

    DATAFILE 3;

    }

  
**Example 8:**

Creating a Long-Term Backup: This example creates a consistent backup of the database that is exempt from the retention policy and tells RMAN to keep the backup for the next year, but not to keep the archived logs necessary to recover it:  

    SHUTDOWN;

    STARTUP MOUNT;

    BACKUP DATABASE UNTIL ‘SYSDATE+365’ NOLOGS;

  
**Example 9:**

Backing Up Files with No Recent Backup: This example backs up all database files and acrchived logs that have not been backed up in the last month.  

    BACKUP DATABASE, ARCHIVELOG ALL NOT BACKED UP SINCE TIME ‘SYSDATE-31’ ;

  
**Example 10:**

This example mounts the database, takes a whole database backup, then opens the database. At the RMAN prompt enter.  

    STARTUP MOUNT;

    BACKUP DATABASE;

    #Now that the backup is complete, open the database

    ALTER DATABASE OPEN;

  
**Example 11:**

To restore the control file to its default location enters the following:  

    STARTUP NOMOUNT;

    RESTORE CONTROLFILE;

    ALTER DATABASE MOUNT;

  
**Example 12:**

This example restores backup archived redo log files from tape that fall within a range of SCNs:  

    RESTORE ARCHIVELOG SCN BETWEEN 500 AND 700;

  
**Example 13:**

This example backs up only archived log 1372 of thread 1 and then deletes it.  

    BACKUP ARCHIVELOG SEQUENCE 1372 DELETE INPUT;

  
**Example 14:**  

This example backs up all archived logs from sequence 288 to sequence 301 on thread 1 and deletes the archived logs after the backup is complete. If the backup fails, the logs are not deleted.

    RUN
    {
    ALLOCATE CHANNEL dev1 DEVICE TYPE sbt;
    BACKUP ARCHIVELOG
    SQUENCE BETWEEN 288 AND 301 THRED 1
    # Delete original archived redo logs after backup completes
    DELETE INPUT;
    }
