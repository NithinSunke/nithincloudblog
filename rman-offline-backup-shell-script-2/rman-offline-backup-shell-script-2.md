---
title: "rman online backup shell script"
author: "Sunke Nithin"
date: 2019-07-03
categories: ["SHELL SCRIPTS"]
tags: []
source: https://nithinsunke.wordpress.com/2019/07/03/rman-offline-backup-shell-script-2/
---

# rman online backup shell script

Script Name: rman_backup.sh

``` wp-block-code
#!/bin/ksh

# +----------------------------------------------------------------------------+
# |                               Nithin Sunke                                 |
# |                          sunkenithin@gmail.com                             |
# |----------------------------------------------------------------------------|
# |               Copyright (c) 2014-2022 Nithin S. All rights reserved.       |
# |----------------------------------------------------------------------------|

# +----------------------------------------------------------------------------+
# |                                                                            |
# |                    DEFINE ALL CUSTOM GLOBAL VARIABLES                      |
# |                                                                            |
# +----------------------------------------------------------------------------+

# ----------------------------
# SCRIPT VERSION
# ----------------------------
VERSION="1.0"

# ----------------------------
# ORGANIZATION INFORMATION
# ----------------------------
# Note: No commas!
# ----------------------------
ORGANIZATION_NAME="Sunketech"

# ----------------------------
# SCRIPT PARAMETER VARIABLES
# ----------------------------
TARGET_DB_NAME=$1
TARGET_SID=$2
TARGET_DBA_USERNAME=$3
TARGET_DBA_PASSWORD=$4
BACKUP_PROFILE=$5
RMAN_RECOVERY_CATALOG=$6
CATALOG_DB_NAME=$7
CATALOG_DBA_USERNAME=$8
CATALOG_DBA_PASSWORD=$9

BACKUP_RMAN_RECOVERY_CATALOG=$6
RMAN_RECOVERY_CATALOG=`echo $RMAN_RECOVERY_CATALOG | tr '[:lower:]' '[:upper:]'`

if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then
    EXPECTED_NUM_SCRIPT_PARAMS=9
elif [[ $RMAN_RECOVERY_CATALOG = "NOCATALOG" ]]; then
    EXPECTED_NUM_SCRIPT_PARAMS=6
else
    RMAN_RECOVERY_CATALOG="INVALID_VALUE"
    EXPECTED_NUM_SCRIPT_PARAMS=6
fi

UNIQUE_SCRIPT_IDENTIFIER=${TARGET_SID}_${TARGET_DBA_USERNAME}

# ----------------------------
# CUSTOM DIRECTORIES
# ----------------------------
CUSTOM_ORACLE_DIR=/backup/scripts
CUSTOM_ORACLE_BIN_DIR=${CUSTOM_ORACLE_DIR}/shell
CUSTOM_ORACLE_LIB_DIR=${CUSTOM_ORACLE_DIR}/lib
CUSTOM_ORACLE_LOG_DIR=${CUSTOM_ORACLE_DIR}/log
CUSTOM_ORACLE_OUT_DIR=${CUSTOM_ORACLE_DIR}/out
CUSTOM_ORACLE_SQL_DIR=${CUSTOM_ORACLE_DIR}/sql
CUSTOM_ORACLE_TEMP_DIR=${CUSTOM_ORACLE_DIR}/temp

# ----------------------------
# SCRIPT NAME VARIABLES
# ----------------------------
SCRIPT_NAME_FULL=$0
SCRIPT_NAME=${SCRIPT_NAME_FULL##*/}
SCRIPT_NAME_NOEXT=${SCRIPT_NAME%.?*}

# ----------------------------
# HOSTNAME VARIABLES
# ----------------------------
HOSTNAME=`hostname`
HOSTNAME_UPPER=`echo $HOSTNAME | tr '[:lower:]' '[:upper:]'`
HOSTNAME_SHORT=${HOSTNAME%%.*}
HOSTNAME_SHORT_UPPER=`echo $HOSTNAME_SHORT | tr '[:lower:]' '[:upper:]'`

# +----------------------------------------------------------------------------+
# |                                                                            |
# |                   DEFINE ALL INTERNAL GLOBAL VARIABLES                     |
# |                                                                            |
# +----------------------------------------------------------------------------+

# ----------------------------
# DATE VARIABLES
# ----------------------------
START_DATE=d`date`
START_DATE_LOG=`date +"%Y%m%d_%H%M%S"`
START_DATE_PRINT=`date +"%m/%d/%Y %r %Z"`
CURRENT_YEAR=`date +"%Y"`;

# ----------------------------
# SHELL PROPERTIES
# ----------------------------
SPROP_SHELL_FLAGS=$-
SPROP_PROCESS_ID=$$
SPROP_NUM_SCRIPT_PARAMS=$#

# ----------------------------
# MISCELLANEOUS VARIABLES
# ----------------------------
HOST_RVAL_SUCCESS=0
HOST_RVAL_WARNING=2
HOST_RVAL_FAILED=2
HIDE_PASSWORD_STRING="xxxxxxxxxxxxx"

# ----------------------------
# LOG AND TEMP FILE VARIABLES
# ----------------------------
LOG_FILE_ARCHIVE_OBSOLETE_DAYS=45
LOG_FILE_NAME=${CUSTOM_ORACLE_LOG_DIR}/${SCRIPT_NAME_NOEXT}_${UNIQUE_SCRIPT_IDENTIFIER}_${HOSTNAME_SHORT_UPPER}_${START_DATE_LOG}.log
LOG_FILE_NAME_NODATE=${CUSTOM_ORACLE_LOG_DIR}/${SCRIPT_NAME_NOEXT}_${UNIQUE_SCRIPT_IDENTIFIER}_${HOSTNAME_SHORT_UPPER}.log
CHECK_SCRIPT_RUNNING_FLAG_FILE=${CUSTOM_ORACLE_TEMP_DIR}/${SCRIPT_NAME_NOEXT}_${UNIQUE_SCRIPT_IDENTIFIER}_${HOSTNAME_SHORT_UPPER}.running
SQL_OUTPUT_TEMP_FILE_NAME=${CUSTOM_ORACLE_TEMP_DIR}/${SCRIPT_NAME_NOEXT}_${UNIQUE_SCRIPT_IDENTIFIER}_${HOSTNAME_SHORT_UPPER}.lst

# +----------------------------------------------------------------------------+
# |                                                                            |
# |                   DEFINE ALL INTERNAL GLOBAL FUNCTIONS                     |
# |                                                                            |
# +----------------------------------------------------------------------------+

function wl {

    typeset -r L_STRING=${1}

    echo "${L_STRING}" >> ${LOG_FILE_NAME}
    echo "${L_STRING}"

    return

}

function printScriptParameterVariables {

    wl " "
    wl "================================================================"
    wl "             PRINT SCRIPT PARAMETER VARIABLES                   "
    wl "================================================================"

    wl " "
    wl "TRACE> TARGET_DB_NAME         set to $TARGET_DB_NAME"
    wl "TRACE> TARGET_SID             set to $TARGET_SID"
    wl "TRACE> TARGET_DBA_USERNAME    set to $TARGET_DBA_USERNAME"
    wl "TRACE> TARGET_DBA_PASSWORD    set to $HIDE_PASSWORD_STRING"
    wl "TRACE> BACKUP_PROFILE         set to $BACKUP_PROFILE"
    wl "TRACE> RMAN_RECOVERY_CATALOG  set to $RMAN_RECOVERY_CATALOG"
    wl "TRACE> CATALOG_DB_NAME        set to $CATALOG_DB_NAME"
    wl "TRACE> CATALOG_DBA_USERNAME   set to $CATALOG_DBA_USERNAME"

    if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then
        wl "TRACE> CATALOG_DBA_PASSWORD   set to $HIDE_PASSWORD_STRING"
    else
        wl "TRACE> CATALOG_DBA_PASSWORD   set to "
    fi

}

function performScriptParameterValidation {

    typeset -r  L_VERSION=${1}
    typeset -r  L_CURRENT_YEAR=${2}

    # -------------------------------------------
    # CHECK FOR VALID RMAN RECOVERY CATALOG VALUE
    # -------------------------------------------
    if [[ $RMAN_RECOVERY_CATALOG = "INVALID_VALUE" ]]; then
        showSignonBanner $L_VERSION $L_CURRENT_YEAR "NOLOG"
        showUsage "NOLOG"
        echo " "
        echo "JMA-0003: Invalid value for script parameter RMAN_RECOVERY_CATALOG ($BACKUP_RMAN_RECOVERY_CATALOG)."
        echo "JMA-0004: RMAN_RECOVERY_CATALOG must be set to either \"catalog\" or \"nocatalog\" (without the double quotes)."
        echo " "
        exit $HOST_RVAL_SUCCESS
    fi

    return

}

function showUsage {

    typeset -r L_WRITE_TO_LOG=${1}
    typeset    L_SHOW

    if [[ $L_WRITE_TO_LOG = "NOLOG" || -z $L_WRITE_TO_LOG ]]; then
        L_SHOW="echo"
    else
        L_SHOW="wl"
    fi

    $L_SHOW " "
    $L_SHOW "Usage: ${SCRIPT_NAME} parameters [optional parameters]"
    $L_SHOW " "
    $L_SHOW "    parameters:  target_db_name"
    $L_SHOW "                 target_sid"
    $L_SHOW "                 target_dba_username"
    $L_SHOW "                 target_dba_password"
    $L_SHOW "                 backup_profile"
    $L_SHOW "                 rman_recovery_catalog [ catalog | nocatalog ]"
    $L_SHOW " "
    $L_SHOW "    optional"
    $L_SHOW "    parameters:  catalog_db_name"
    $L_SHOW "                 catalog_dba_username"
    $L_SHOW "                 catalog_dba_password"
    $L_SHOW " "

    return

}

function showSignonBanner {

    typeset -r L_VERSION=${1}
    typeset -r L_CURRENT_YEAR=${2}
    typeset -r L_WRITE_TO_LOG=${3}
    typeset    L_SHOW

    if [[ $L_WRITE_TO_LOG = "NOLOG" || -z $L_WRITE_TO_LOG ]]; then
        L_SHOW="echo"
    else
        L_SHOW="wl"
    fi

    $L_SHOW " "
    $L_SHOW "${SCRIPT_NAME} - Version ${L_VERSION}"
    $L_SHOW "Copyright (c) 2014-${L_CURRENT_YEAR} Nithin Sunke. All rights reserved."
    $L_SHOW " "

    return

}

function startLogging {

    wl "+=========================================================================+"
    wl "|                                                                         |"
    wl "|                               START TIME                                |"
    wl "|                                                                         |"
    wl "|                      $START_DATE                       |"
    wl "|                                                                         |"
    wl "+=========================================================================+"

    return

}

function stopLogging {

    END_DATE=`${DATE_BIN}`
    wl " "
    wl "+=========================================================================+"
    wl "|                                                                         |"
    wl "|                               FINISH TIME                               |"
    wl "|                                                                         |"
    wl "|                       $END_DATE                      |"
    wl "|                                                                         |"
    wl "+=========================================================================+"

    return

}

function initializeScript {

    typeset -ru L_FIRST_PARAMETER=${1}
    typeset -r  L_VERSION=${2}
    typeset -r  L_CURRENT_YEAR=${3}

    # ----------------------------------------
    # CHECK IF USER ASKED FOR HELP
    # ----------------------------------------
    if [[ $L_FIRST_PARAMETER = "-H" || $L_FIRST_PARAMETER = "-HELP" || $L_FIRST_PARAMETER = "--HELP" || -z $L_FIRST_PARAMETER ]]; then
        showSignonBanner $L_VERSION $L_CURRENT_YEAR "NOLOG"
        showUsage "NOLOG"
        exit $HOST_RVAL_SUCCESS
    fi

    # ----------------------------------------
    # VERIFY CORRECT NUMBER OF PARAMETERS
    # ----------------------------------------
    if (( $SPROP_NUM_SCRIPT_PARAMS != $EXPECTED_NUM_SCRIPT_PARAMS )); then
        showSignonBanner $L_VERSION $L_CURRENT_YEAR "NOLOG"
        showUsage "NOLOG"
        echo " "
        echo "JMA-0001: Number of script parameters passed to this script = $SPROP_NUM_SCRIPT_PARAMS."
        echo "JMA-0002: Number of expected script parameters to this script = $EXPECTED_NUM_SCRIPT_PARAMS."
        echo " "
        exit $HOST_RVAL_SUCCESS
    fi

    # --------------------------------------------------
    # PERFORM SCRIPT PARAMETER VALIDATION (if necessary)
    # --------------------------------------------------
    performScriptParameterValidation $L_VERSION $L_CURRENT_YEAR

    # ----------------------------------------
    # CLEAN LOG FILE AND ENVIRONMENT VARIABLES
    # ----------------------------------------
    rm -f ${LOG_FILE_NAME}
    NEW_ORACLE_HOME="NO_ORACLE_HOME_FOUND"
    ERRORS="NO"
    unset TWO_TASK

    # ----------------------------------------
    # INITIALIZE LOG FILE
    # ----------------------------------------
    startLogging

    # ----------------------------------------
    # DISPLAY SIGN ON BANNER
    # ----------------------------------------
    showSignonBanner $L_VERSION $L_CURRENT_YEAR "LOG"

    # ----------------------------------------
    # PRINT SCRIPT PARAMETER VARIABLES
    # ----------------------------------------
    printScriptParameterVariables

    return

}

function source_env {

. /home/oracle/profile.$TARGET_DB_NAME

}

function backupCurrentLogFile {

    cp -vf ${LOG_FILE_NAME} ${LOG_FILE_NAME_NODATE}

    wl " "
    wl "TRACE> Copied ${LOG_FILE_NAME} to ${LOG_FILE_NAME_NODATE}"

    return

}

function removeScriptRunFlagFile {

    rm -vf ${CHECK_SCRIPT_RUNNING_FLAG_FILE}

    wl " "
    wl "TRACE> Removed script run flag file (${CHECK_SCRIPT_RUNNING_FLAG_FILE})"

    return

}

function exitSuccess {

    typeset L_SEVERITY=${1}
    typeset L_UNIQUE_SCRIPT_IDENTIFIER=${2}

    wl " "
    wl "TRACE> +----------------------------------------------+"
    wl "TRACE> |                  SUCCESSFUL                  |"
    wl "TRACE> +----------------------------------------------+"
    wl " "
    wl "TRACE> Exiting script (${HOST_RVAL_SUCCESS})."
    wl " "

    removeScriptRunFlagFile
    stopLogging

    backupCurrentLogFile

    exit ${HOST_RVAL_SUCCESS}

}

function exitWarning {

    typeset L_SEVERITY=${1}
    typeset L_UNIQUE_SCRIPT_IDENTIFIER=${2}

    wl " "
    wl "TRACE> +----------------------------------------------+"
    wl "TRACE> |       !!!!!!!!    WARNING    !!!!!!!!        |"
    wl "TRACE> +----------------------------------------------+"
    wl " "
    wl "TRACE> Exiting script (${HOST_RVAL_WARNING})."
    wl " "

    removeScriptRunFlagFile
    stopLogging

    backupCurrentLogFile

    exit ${HOST_RVAL_WARNING}

}

function exitFailed {

    typeset L_SEVERITY=${1}
    typeset L_UNIQUE_SCRIPT_IDENTIFIER=${2}

    wl " "
    wl "TRACE> +----------------------------------------------+"
    wl "TRACE> |    !!!!!!!!    CRITICAL ERROR    !!!!!!!!    |"
    wl "TRACE> +----------------------------------------------+"
    wl " "
    wl "TRACE> Exiting script (${HOST_RVAL_FAILED})."
    wl " "

    if [[ $L_SEVERITY = "RUNNING" ]]; then
      wl " "
      wl "TRACE> Script was found to be already running."
      wl "TRACE> Do not remove the script run flag file."
    else
      removeScriptRunFlagFile
    fi

    # showUsage "LOG"
    stopLogging

    backupCurrentLogFile

    exit ${HOST_RVAL_FAILED}

}

function checkScriptAlreadyRunning {

    typeset -r L_SCRIPT_NAME=${1}
    typeset -r L_UNIQUE_SCRIPT_IDENTIFIER=${2}
    typeset    L_COMMAND

    wl " "
    wl "TRACE> Check that this script (${L_SCRIPT_NAME}) is not already running on this host."

    wl " "
    wl "TRACE> Looking for script run flag file (${CHECK_SCRIPT_RUNNING_FLAG_FILE})."

    if [ -f $CHECK_SCRIPT_RUNNING_FLAG_FILE ]; then
        wl " "
        wl "TRACE> WARNING: Found ${L_SCRIPT_NAME} already running on this host. Exiting script."
        exitFailed "RUNNING" ${L_UNIQUE_SCRIPT_IDENTIFIER}
    else
        wl " "
        wl "TRACE> Did not find this script (${L_SCRIPT_NAME}) already running on this host. Setting run flag and continuing script..."
        touch $CHECK_SCRIPT_RUNNING_FLAG_FILE
    fi
    wl " "

    return

}

function verifyOSUserLogin {

    typeset -r L_CHECK_USER_NAME=${1}
    typeset -r L_REQUIRED=${2}
    typeset -r L_UNIQUE_SCRIPT_IDENTIFIER=${3}
    typeset    L_UID

    # L_UID=`/usr/bin/id|awk -F\( '{print $2}'|awk -F\) '{print $1}'`
    L_UID=`id|awk -F\( '{print $2}'|awk -F\) '{print $1}'`

    wl ""
    wl "TRACE> OS user logged in (${L_UID})."

    if [[ ${L_REQUIRED} == "TRUE" ]]; then

        if [[ ${L_UID} != "${L_CHECK_USER_NAME}" ]]; then
            wl " "
            wl "TRACE> You must be logged in as the (${L_CHECK_USER_NAME}) OS user to run this script."
            wl "TRACE> Log in to the machine as (${L_CHECK_USER_NAME}) and restart execution of this script."
            exitFailed "FAILED" ${L_UNIQUE_SCRIPT_IDENTIFIER}
        else
            wl " "
            wl "TRACE> Successfully logged in as (${L_CHECK_USER_NAME})."
        fi

    else
        wl " "
        wl "TRACE> OS user is not required to be logged in as (${L_CHECK_USER_NAME})."
    fi

    return

}

function verifyOracleSID {

    typeset -r L_SID_NAME=${1}
    typeset -r L_UNIQUE_SCRIPT_IDENTIFIER=${2}
    typeset    L_COMMAND

    wl " "
    wl "TRACE> Check that the Oracle instance (${L_SID_NAME}) is up."

    L_COMMAND="ps -ef | grep \"ora_smon_$L_SID_NAME\$\" | grep -v 'grep'"
    wl "TRACE> ${L_COMMAND}"
    wl " "
    ps -ef | grep "ora_smon_$L_SID_NAME$" | grep -v 'grep'

    if (( $? == 0 )); then
        wl " "
        wl "TRACE> The Oracle instance (${L_SID_NAME}) IS running on this host."
    else
        wl " "
        wl "TRACE> The Oracle instance (${L_SID_NAME}) IS NOT running on this host."
        wl "TRACE> The Oracle instance (${L_SID_NAME}) must be running on this host to continue."
        exitFailed "FAILED" ${L_UNIQUE_SCRIPT_IDENTIFIER}
    fi

    return

}

function verifyTNSConnectString {

    typeset -r L_DB_NAME=${1}
    typeset -r L_UNIQUE_SCRIPT_IDENTIFIER=${2}

    wl " "
    wl "TRACE> Check that the Oracle TNS connect string (${L_DB_NAME}) is valid."
    wl " "

    $ORACLE_HOME/bin/tnsping ${L_DB_NAME}

    if (( $? == 0 )); then
        wl " "
        wl "TRACE> The TNS service name ($L_DB_NAME) IS valid."
    else
        wl " "
        wl "TRACE> The TNS service name ($L_DB_NAME) IS NOT valid."
        exitFailed "FAILED" ${L_UNIQUE_SCRIPT_IDENTIFIER}
    fi

    return

}

function verifyDatabaseLoginCredentials {

    typeset -r L_DB_NAME=${1}
    typeset -r L_DBA_USERNAME=${2}
    typeset -r L_DBA_PASSWORD=${3}
    typeset -r L_SYSDBA_PRIVS=${4}
    typeset -r L_UNIQUE_SCRIPT_IDENTIFIER=${5}
    typeset    L_SYSDBA_PRIVS_TXT
    typeset -i L_EXIT_STATUS

    if [[ $L_SYSDBA_PRIVS = "SYSDBA" ]]; then
        L_SYSDBA_PRIVS_TXT=" AS SYSDBA"
    else
        L_SYSDBA_PRIVS_TXT=""
    fi

    wl " "
    wl "TRACE> Test log in credentials to database (${L_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_DB_NAME}${L_SYSDBA_PRIVS_TXT})."
    wl " "

    $ORACLE_HOME/bin/sqlplus -s /nolog <<EOF
      WHENEVER OSERROR EXIT 9
      WHENEVER SQLERROR EXIT SQL.SQLCODE
      SPOOL ${SQL_OUTPUT_TEMP_FILE_NAME} REPLACE
      CONNECT ${L_DBA_USERNAME}/${L_DBA_PASSWORD}@${L_DB_NAME} ${L_SYSDBA_PRIVS_TXT}
      SET HEAD OFF
      SET LINESIZE 145
      SET PAGESIZE 9000
      COLUMN USERNAME FORMAT A10
      COLUMN PROGRAM FORMAT A45
      COLUMN MACHINE FORMAT A30
      SELECT 'SQL*TRACE> Successfully logged in to the database (${L_DB_NAME}${L_SYSDBA_PRIVS_TXT}) as the [' || lower(user) || '] user.' FROM dual;
      SPOOL OFF
EOF

    L_EXIT_STATUS=$?
    wl "TRACE> SQL*Plus exit status ($L_EXIT_STATUS)."

    egrep 'ORA-|PLS-|SP2-' ${SQL_OUTPUT_TEMP_FILE_NAME}

    if (( $? == 0 ))
    then
        wl " "
        wl "TRACE> Database credentials for (${L_DB_NAME}${L_SYSDBA_PRIVS_TXT}) ARE NOT valid."
        exitFailed "FAILED" ${L_UNIQUE_SCRIPT_IDENTIFIER}
    else
        wl " "
        wl "TRACE> Database credentials for (${L_DB_NAME}${L_SYSDBA_PRIVS_TXT}) are valid."
        wl " "
        wl "TRACE> Removing temporary SQL output file ($SQL_OUTPUT_TEMP_FILE_NAME)."
        rm -f $SQL_OUTPUT_TEMP_FILE_NAME
    fi

    return

}

# +----------------------------------------------------------------------------+
# |                                                                            |
# |                    DEFINE ALL CUSTOM GLOBAL FUNCTIONS                      |
# |                                                                            |
# +----------------------------------------------------------------------------+

function performRMAN_FULL_Backup {

    typeset -r L_TARGET_DB_NAME=${1}
    typeset -r L_TARGET_DBA_USERNAME=${2}
    typeset -r L_TARGET_DBA_PASSWORD=${3}
    typeset -r L_CATALOG_DB_NAME=${4}
    typeset -r L_CATALOG_DBA_USERNAME=${5}
    typeset -r L_CATALOG_DBA_PASSWORD=${6}
    typeset    L_RMAN_CONNECT_TARGET
    typeset    L_RMAN_CONNECT_CATALOG

    wl " "
    wl "TRACE> +---------------------------------+"
    wl "TRACE> | RMAN Backup Profile Description |"
    wl "TRACE> |------------------------------------------------------------------------+"
    wl "TRACE> | * Full database backup + archivelog delete input to disk.              |"
    wl "TRACE> | * Compatibility: 10g / 11g / 12c / 18c / 19c /20c                      |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl " "

    # ---------------------------------------------
    # SET LOG IN CREDENTIALS
    # ---------------------------------------------
    L_RMAN_CONNECT_TARGET="CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${L_TARGET_DBA_PASSWORD}@${L_TARGET_DB_NAME}"
    wl "TRACE> [TARGET] CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_TARGET_DB_NAME}"

    if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then
        L_RMAN_CONNECT_CATALOG="CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${L_CATALOG_DBA_PASSWORD}@${L_CATALOG_DB_NAME}"
        wl "TRACE> [CATALOG] CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_CATALOG_DB_NAME}"
    else
        L_RMAN_CONNECT_CATALOG=""
        wl "TRACE> [CATALOG] No recovery catalog credentials defined."
    fi

    # create the directory structure
      mkdir -p /backup/${ORACLE_SID}/full

    # ---------------------------------------------
    # PERFORM FULL RMAN BACKUP
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        @/backup/scripts/cmd/configuration_setup_full.cmd

        run {
                report schema;
                crosscheck backup of database;
                crosscheck backup of archivelog all;
                crosscheck backup of controlfile;
                crosscheck backup of spfile;
                crosscheck archivelog all;
                delete noprompt force expired backup;
                delete noprompt force expired archivelog all;
                delete noprompt force expired copy;
                delete noprompt force obsolete;
        }

        run {
                backup database plus archivelog delete input;
        }

        allocate channel for maintenance type disk;
        delete noprompt force obsolete;
        report need backup;
        report unrecoverable;

        exit;
EOF

    return

}

function performRMAN_INC0_Backup {

    typeset -r L_TARGET_DB_NAME=${1}
    typeset -r L_TARGET_DBA_USERNAME=${2}
    typeset -r L_TARGET_DBA_PASSWORD=${3}
    typeset -r L_CATALOG_DB_NAME=${4}
    typeset -r L_CATALOG_DBA_USERNAME=${5}
    typeset -r L_CATALOG_DBA_PASSWORD=${6}
    typeset -i L_RMAN_INC_LEVEL
    typeset    L_RMAN_CONNECT_TARGET
    typeset    L_RMAN_CONNECT_CATALOG

    wl " "
    wl "TRACE> +---------------------------------+"
    wl "TRACE> | RMAN Backup Profile Description |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl "TRACE> | * Incremental database backup (level 0) to disk                        |"
    wl "TRACE> | * Delete archivelog files older then N days.                           |"
    wl "TRACE> | * Backup current control file and SPFILE to disk.                      |"
    wl "TRACE> | * Compatibility: 10g / 11g                                             |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl " "

    # ---------------------------------------------
    # SET LOG IN CREDENTIALS
    # ---------------------------------------------
    L_RMAN_CONNECT_TARGET="CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${L_TARGET_DBA_PASSWORD}@${L_TARGET_DB_NAME}"
    wl "TRACE> [TARGET] CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_TARGET_DB_NAME}"

    if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then
        L_RMAN_CONNECT_CATALOG="CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${L_CATALOG_DBA_PASSWORD}@${L_CATALOG_DB_NAME}"
        wl "TRACE> [CATALOG] CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_CATALOG_DB_NAME}"
    else
        L_RMAN_CONNECT_CATALOG=""
        wl "TRACE> [CATALOG] No recovery catalog credentials defined."
    fi

    # create directory structure
    mkdir -p /backup/${ORACLE_SID}/inc0

    # ---------------------------------------------
    # REPORT, CROSSCHECK, AND DELETE EXPIRED
    # BACKUPS
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        @/backup/scripts/cmd/configuration_setup_inc0.cmd

        run {
                report schema;
                crosscheck backup of database;
                crosscheck backup of archivelog all;
                crosscheck backup of controlfile;
                crosscheck backup of spfile;
                crosscheck archivelog all;
                delete noprompt force expired backup;
                delete noprompt force expired archivelog all;
                delete noprompt force expired copy;
                delete noprompt force obsolete;
        }

        exit;
EOF

    # ---------------------------------------------
    # PERFORM INCREMENTAL RMAN BACKUP OF DATABASE
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        run {
                backup incremental level=0 database;
                sql 'alter system switch logfile';
                backup archivelog all not backed up;
                delete noprompt force archivelog all completed before 'sysdate-30';
                backup current controlfile;
                backup spfile;
        }

        allocate channel for maintenance type disk;
        delete noprompt obsolete;
        report need backup;
        report unrecoverable;

        exit;
EOF

    return

}

function performRMAN_INC1_Dif_Backup {

    typeset -r L_TARGET_DB_NAME=${1}
    typeset -r L_TARGET_DBA_USERNAME=${2}
    typeset -r L_TARGET_DBA_PASSWORD=${3}
    typeset -r L_CATALOG_DB_NAME=${4}
    typeset -r L_CATALOG_DBA_USERNAME=${5}
    typeset -r L_CATALOG_DBA_PASSWORD=${6}
    typeset -i L_RMAN_INC_LEVEL
    typeset    L_RMAN_CONNECT_TARGET
    typeset    L_RMAN_CONNECT_CATALOG

    wl " "
    wl "TRACE> +---------------------------------+"
    wl "TRACE> | RMAN Backup Profile Description |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl "TRACE> | * Incremental Differential database backup (level 1) to disk                        |"
    wl "TRACE> | * Delete archivelog files older then N days.                           |"
    wl "TRACE> | * Backup current control file and SPFILE to disk.                      |"
    wl "TRACE> | * Compatibility: 10g / 11g                                             |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl " "

    # ---------------------------------------------
    # SET LOG IN CREDENTIALS
    # ---------------------------------------------
    L_RMAN_CONNECT_TARGET="CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${L_TARGET_DBA_PASSWORD}@${L_TARGET_DB_NAME}"
    wl "TRACE> [TARGET] CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_TARGET_DB_NAME}"

    if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then
        L_RMAN_CONNECT_CATALOG="CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${L_CATALOG_DBA_PASSWORD}@${L_CATALOG_DB_NAME}"
        wl "TRACE> [CATALOG] CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_CATALOG_DB_NAME}"
    else
        L_RMAN_CONNECT_CATALOG=""
        wl "TRACE> [CATALOG] No recovery catalog credentials defined."
    fi

    # create directory structure
    mkdir -p /backup/${ORACLE_SID}/inc1d

    # ---------------------------------------------
    # REPORT, CROSSCHECK, AND DELETE EXPIRED
    # BACKUPS
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        @/backup/scripts/cmd/configuration_setup_inc1d.cmd

        run {
                report schema;
                crosscheck backup of database;
                crosscheck backup of archivelog all;
                crosscheck backup of controlfile;
                crosscheck backup of spfile;
                crosscheck archivelog all;
                delete noprompt force expired backup;
                delete noprompt force expired archivelog all;
                delete noprompt force expired copy;
                delete noprompt force obsolete;
        }

        exit;
EOF

    # ---------------------------------------------
    # PERFORM INCREMENTAL RMAN BACKUP OF DATABASE
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        run {
                backup incremental level=1 database;
                sql 'alter system switch logfile';
                backup archivelog all not backed up;
                delete noprompt force archivelog all completed before 'sysdate-30';
                backup current controlfile;
                backup spfile;
        }

                allocate channel for maintenance type disk;
                delete noprompt force obsolete;

        run {
            report need backup;
            report unrecoverable;
        }

        exit;
EOF

    return

}

function performRMAN_INC1_CUM_Backup {

    typeset -r L_TARGET_DB_NAME=${1}
    typeset -r L_TARGET_DBA_USERNAME=${2}
    typeset -r L_TARGET_DBA_PASSWORD=${3}
    typeset -r L_CATALOG_DB_NAME=${4}
    typeset -r L_CATALOG_DBA_USERNAME=${5}
    typeset -r L_CATALOG_DBA_PASSWORD=${6}
    typeset -i L_RMAN_INC_LEVEL
    typeset    L_RMAN_CONNECT_TARGET
    typeset    L_RMAN_CONNECT_CATALOG

    wl " "
    wl "TRACE> +---------------------------------+"
    wl "TRACE> | RMAN Backup Profile Description |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl "TRACE> | * IncrementalI CUMMULIATIVE database backup (level 0) to disk                        |"
    wl "TRACE> | * Delete archivelog files older then N days.                           |"
    wl "TRACE> | * Backup current control file and SPFILE to disk.                      |"
    wl "TRACE> | * Compatibility: 10g / 11g                                             |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl " "

    # ---------------------------------------------
    # SET LOG IN CREDENTIALS
    # ---------------------------------------------
    L_RMAN_CONNECT_TARGET="CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${L_TARGET_DBA_PASSWORD}@${L_TARGET_DB_NAME}"
    wl "TRACE> [TARGET] CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_TARGET_DB_NAME}"

    if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then
        L_RMAN_CONNECT_CATALOG="CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${L_CATALOG_DBA_PASSWORD}@${L_CATALOG_DB_NAME}"
        wl "TRACE> [CATALOG] CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_CATALOG_DB_NAME}"
    else
        L_RMAN_CONNECT_CATALOG=""
        wl "TRACE> [CATALOG] No recovery catalog credentials defined."
    fi

    # create directory structure
    mkdir -p /backup/${ORACLE_SID}/inc1c

    # ---------------------------------------------
    # REPORT, CROSSCHECK, AND DELETE EXPIRED
    # BACKUPS
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        @/backup/scripts/cmd/configuration_setup_inc1c.cmd

        run {
                report schema;
                crosscheck backup of database;
                crosscheck backup of archivelog all;
                crosscheck backup of controlfile;
                crosscheck backup of spfile;
                crosscheck archivelog all;
                delete noprompt force expired backup;
                delete noprompt force expired archivelog all;
                delete noprompt force expired copy;
                delete noprompt force obsolete;
        }

        exit;
EOF

    # ---------------------------------------------
    # PERFORM INCREMENTAL RMAN BACKUP OF DATABASE
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        run {
                backup incremental level=1 CUMULATIVE  database;
                sql 'alter system switch logfile';
                backup archivelog all not backed up;
                delete noprompt force archivelog all completed before 'sysdate-30';
                backup current controlfile;
                backup spfile;
        }

                allocate channel for maintenance type disk;
                delete noprompt force obsolete;

        run {
            report need backup;
            report unrecoverable;
        }

        exit;
EOF

    return

}

function performRMAN_Arch_Backup {

    typeset -r L_TARGET_DB_NAME=${1}
    typeset -r L_TARGET_DBA_USERNAME=${2}
    typeset -r L_TARGET_DBA_PASSWORD=${3}
    typeset -r L_CATALOG_DB_NAME=${4}
    typeset -r L_CATALOG_DBA_USERNAME=${5}
    typeset -r L_CATALOG_DBA_PASSWORD=${6}
    typeset -i L_RMAN_INC_LEVEL
    typeset    L_RMAN_CONNECT_TARGET
    typeset    L_RMAN_CONNECT_CATALOG

    wl " "
    wl "TRACE> +---------------------------------+"
    wl "TRACE> | RMAN Backup Profile Description |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl "TRACE> | * Archivelog  backup  to disk                                          |"
    wl "TRACE> | * Delete archivelog files older then N days.                           |"
    wl "TRACE> | * Backup current control file and SPFILE to disk.                      |"
    wl "TRACE> | * Compatibility: 10g / 11g                                             |"
    wl "TRACE> +------------------------------------------------------------------------+"
    wl " "

    # ---------------------------------------------
    # SET LOG IN CREDENTIALS
    # ---------------------------------------------
    L_RMAN_CONNECT_TARGET="CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${L_TARGET_DBA_PASSWORD}@${L_TARGET_DB_NAME}"
    wl "TRACE> [TARGET] CONNECT TARGET ${L_TARGET_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_TARGET_DB_NAME}"

    if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then
        L_RMAN_CONNECT_CATALOG="CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${L_CATALOG_DBA_PASSWORD}@${L_CATALOG_DB_NAME}"
        wl "TRACE> [CATALOG] CONNECT CATALOG ${L_CATALOG_DBA_USERNAME}/${HIDE_PASSWORD_STRING}@${L_CATALOG_DB_NAME}"
    else
        L_RMAN_CONNECT_CATALOG=""
        wl "TRACE> [CATALOG] No recovery catalog credentials defined."
    fi

   # create directory structure
   mkdir -p /backup/${ORACLE_SID}/arch

    # ---------------------------------------------
    # REPORT, CROSSCHECK, AND DELETE EXPIRED
    # BACKUPS
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        @/backup/scripts/cmd/configuration_setup_arch.cmd

        run {
                report schema;
                crosscheck backup of database;
                crosscheck backup of archivelog all;
                crosscheck backup of controlfile;
                crosscheck backup of spfile;
                crosscheck archivelog all;
                delete noprompt force expired backup;
                delete noprompt force expired archivelog all;
                delete noprompt force expired copy;
                delete noprompt force obsolete;
        }

        exit;
EOF

    # ---------------------------------------------
    # PERFORM INCREMENTAL RMAN BACKUP OF DATABASE
    # ---------------------------------------------
    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME
        $L_RMAN_CONNECT_TARGET
        $L_RMAN_CONNECT_CATALOG

        run {
                sql 'alter system switch logfile';
                backup archivelog all not backed up delete input;
                delete noprompt force archivelog all completed before 'sysdate-30';
                backup current controlfile;
                backup spfile;
        }

                allocate channel for maintenance type disk;
                delete noprompt force obsolete;

        run {
            report need backup;
            report unrecoverable;
        }

        exit;
EOF

    return

}

# +----------------------------------------------------------------------------+
# |                                                                            |
# |                            SCRIPT STARTS HERE                              |
# |                                                                            |
# +----------------------------------------------------------------------------+

initializeScript "${1}" $VERSION $CURRENT_YEAR

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| VERIFY OS LOGIN.                                                        |"
wl "+-------------------------------------------------------------------------+"

verifyOSUserLogin "oracle" "FALSE" $UNIQUE_SCRIPT_IDENTIFIER

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| VERIFY AN INSTANCE OF THIS SCRIPT IS NOT ALREADY RUNNING.               |"
wl "+-------------------------------------------------------------------------+"
checkScriptAlreadyRunning $SCRIPT_NAME $UNIQUE_SCRIPT_IDENTIFIER

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "|  source environment.....                                             |"
wl "+-------------------------------------------------------------------------+"
source_env

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| DISPLAY ALL SCRIPT ENVIRONMENT VARIABLES.                               |"
wl "+-------------------------------------------------------------------------+"

wl " "
wl "================================================================"
wl "                 GLOBAL SCRIPT VARIABLES                        "
wl "================================================================"
wl "ORGANIZATION NAME                    : $ORGANIZATION_NAME"
wl "SCRIPT                               : $SCRIPT_NAME"
wl "VERSION                              : $VERSION"
wl "START DATE/TIME                      : $START_DATE"
wl "SHELL ACCESS                         : $SPROP_SHELL_ACCESS"
wl "SHELL FLAGS                          : $SPROP_SHELL_FLAGS"
wl "PROCESS ID                           : $SPROP_PROCESS_ID"
wl "# OF SCRIPT PARAMETERS               : $SPROP_NUM_SCRIPT_PARAMS"
wl "# OF EXPECTED SCRIPT PARAMETERS      : $EXPECTED_NUM_SCRIPT_PARAMS"
wl "UNIQUE_SCRIPT_IDENTIFIER             : $UNIQUE_SCRIPT_IDENTIFIER"
wl "CHECK_SCRIPT_RUNNING_FLAG_FILE       : $CHECK_SCRIPT_RUNNING_FLAG_FILE"
wl "HOST_NAME                            : $HOSTNAME"
wl "HOST_NAME (UPPER)                    : $HOSTNAME_UPPER"
wl "HOST_NAME (SHORT)                    : $HOSTNAME_SHORT"
wl "HOST_NAME (SHORT/UPPER)              : $HOSTNAME_SHORT_UPPER"
wl "ORACLE_HOME                          : $ORACLE_HOME"
wl "LOG_FILE_NAME                        : $LOG_FILE_NAME"
wl "LOG_FILE_NAME_NODATE                 : $LOG_FILE_NAME_NODATE"
wl "LOG_FILE_ARCHIVE_OBSOLETE_DAYS       : $LOG_FILE_ARCHIVE_OBSOLETE_DAYS"

wl " "
wl "==========================================================="
wl "                  COMMAND-LINE ARGUMENTS                   "
wl "==========================================================="
wl "TARGET_DB_NAME         (P1)          : $TARGET_DB_NAME"
wl "TARGET_SID             (P2)          : $TARGET_SID"
wl "TARGET_DBA_USERNAME    (P3)          : $TARGET_DBA_USERNAME"
wl "TARGET_DBA_PASSWORD    (P4)          : $HIDE_PASSWORD_STRING"
wl "BACKUP_PROFILE         (P5)          : $BACKUP_PROFILE"
wl "RMAN_RECOVERY_CATALOG  (P6)          : $RMAN_RECOVERY_CATALOG"
wl "CATALOG_DB_NAME        (P7)          : $CATALOG_DB_NAME"
wl "CATALOG_DBA_USERNAME   (P8)          : $CATALOG_DBA_USERNAME"
if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then
    wl "CATALOG_DBA_PASSWORD   (P9)          : $HIDE_PASSWORD_STRING"
else
    wl "CATALOG_DBA_PASSWORD   (P9)          : "
fi

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| THIS SCRIPT IS TO BE RUN FROM THE DATABASE SERVER HOSTING THE TARGET    |"
wl "| DATABASE. VERIFY ORACLE INSTANCE IS UP AND RUNNING ON THIS HOST.        |"
wl "+-------------------------------------------------------------------------+"

verifyOracleSID $TARGET_SID $UNIQUE_SCRIPT_IDENTIFIER

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| VERIFY THE ORACLE TNS CONNECT STRING IS VALID TO THE TARGET DATABASE.   |"
wl "+-------------------------------------------------------------------------+"

verifyTNSConnectString $TARGET_DB_NAME $UNIQUE_SCRIPT_IDENTIFIER

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| VERIFY LOG IN CREDENTIALS TO THE TARGET DATABASE.                       |"
wl "+-------------------------------------------------------------------------+"

verifyDatabaseLoginCredentials $TARGET_DB_NAME $TARGET_DBA_USERNAME $TARGET_DBA_PASSWORD "SYSDBA" $UNIQUE_SCRIPT_IDENTIFIER

# +----------------------------------------------------------------------------+
# |                                                                            |
# |                        CUSTOM SCRIPT TASKS (BEGIN)                         |
# |                                                                            |
# +----------------------------------------------------------------------------+

if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
    wl " "
    wl "+-------------------------------------------------------------------------+"
    wl "| ${DATE_PRINT_LOG}                                                     |"
    wl "|-------------------------------------------------------------------------|"
    wl "| VERIFY THE ORACLE TNS CONNECT STRING IS VALID TO THE CATALOG DATABASE.  |"
    wl "+-------------------------------------------------------------------------+"

    verifyTNSConnectString $CATALOG_DB_NAME $UNIQUE_SCRIPT_IDENTIFIER

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
    wl " "
    wl "+-------------------------------------------------------------------------+"
    wl "| ${DATE_PRINT_LOG}                                                     |"
    wl "|-------------------------------------------------------------------------|"
    wl "| VERIFY LOG IN CREDENTIALS TO THE CATALOG DATABASE.                      |"
    wl "+-------------------------------------------------------------------------+"

    verifyDatabaseLoginCredentials $CATALOG_DB_NAME $CATALOG_DBA_USERNAME $CATALOG_DBA_PASSWORD "NOSYSDBA" $UNIQUE_SCRIPT_IDENTIFIER

fi

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| PERFORM RMAN DATABASE BACKUP PROFILE.                                   |"
wl "+-------------------------------------------------------------------------+"

wl " "
wl "TRACE> RMAN Backup Profile ${BACKUP_PROFILE}"

case ${BACKUP_PROFILE} in
    1)  performRMAN_FULL_Backup $TARGET_DB_NAME $TARGET_DBA_USERNAME $TARGET_DBA_PASSWORD $CATALOG_DB_NAME $CATALOG_DBA_USERNAME $CATALOG_DBA_PASSWORD
        ;;
    2)  performRMAN_INC0_Backup $TARGET_DB_NAME $TARGET_DBA_USERNAME $TARGET_DBA_PASSWORD $CATALOG_DB_NAME $CATALOG_DBA_USERNAME $CATALOG_DBA_PASSWORD
        ;;
    3)  performRMAN_INC1_CUM_Backup $TARGET_DB_NAME $TARGET_DBA_USERNAME $TARGET_DBA_PASSWORD $CATALOG_DB_NAME $CATALOG_DBA_USERNAME $CATALOG_DBA_PASSWORD
        ;;
    4)  performRMAN_INC1_Dif_Backup $TARGET_DB_NAME $TARGET_DBA_USERNAME $TARGET_DBA_PASSWORD $CATALOG_DB_NAME $CATALOG_DBA_USERNAME $CATALOG_DBA_PASSWORD
        ;;
    5)  performRMAN_Arch_Backup $TARGET_DB_NAME $TARGET_DBA_USERNAME $TARGET_DBA_PASSWORD $CATALOG_DB_NAME $CATALOG_DBA_USERNAME $CATALOG_DBA_PASSWORD
        ;;
    *)  wl " "
        wl "TRACE> JMA-1001: Invalid RMAN Backup Profile (${BACKUP_PROFILE})."
        wl "TRACE> JMA-1002: Restart this script with a valid Backup Profile value."
        ;;
esac

# +----------------------------------------------------------------------------+
# |                                                                            |
# |                       CUSTOM SCRIPT TASKS ( END )                          |
# |                                                                            |
# +----------------------------------------------------------------------------+

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| SCAN LOG FILE FOR EXCEPTIONS. IGNORE KNOWN EXCEPTIONS.                  |"
wl "+-------------------------------------------------------------------------+"

egrep 'ORA-|JMA-|RMAN-' $LOG_FILE_NAME | egrep -v 'RMAN-20242|RMAN-00571|RMAN-00569|RMAN-03002'

if (( $? == 0 ))
then
    wl " "
    wl "+----------------------------------------------+"
    wl "| !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!|"
    wl "|                                              |"
    wl "|   --->        ERRORS WERE FOUND       <---   |"
    wl "|                                              |"
    wl "| !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!|"
    wl "+----------------------------------------------+"
    ERRORS="YES"
else
    wl " "
    wl "TRACE> No exceptions were found."
    ERRORS="NO"
fi

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| REMOVING OBSOLETE SCRIPT LOG FILES (greater than $LOG_FILE_ARCHIVE_OBSOLETE_DAYS days old)"
wl "|   (${CUSTOM_ORACLE_LOG_DIR}/${SCRIPT_NAME_NOEXT}_${UNIQUE_SCRIPT_IDENTIFIER}_${HOSTNAME_SHORT_UPPER}_*.log)"
wl "+-------------------------------------------------------------------------+"

find ${CUSTOM_ORACLE_LOG_DIR}/ -name "${SCRIPT_NAME_NOEXT}_${UNIQUE_SCRIPT_IDENTIFIER}_${HOSTNAME_SHORT_UPPER}_*.log" -mtime +${LOG_FILE_ARCHIVE_OBSOLETE_DAYS} -exec ls -l {} \; | tee -a $LOG_FILE_NAME
find ${CUSTOM_ORACLE_LOG_DIR}/ -name "${SCRIPT_NAME_NOEXT}_${UNIQUE_SCRIPT_IDENTIFIER}_${HOSTNAME_SHORT_UPPER}_*.log" -mtime +${LOG_FILE_ARCHIVE_OBSOLETE_DAYS} -exec rm -rf {} \; | tee -a $LOG_FILE_NAME

DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`
wl " "
wl "+-------------------------------------------------------------------------+"
wl "| ${DATE_PRINT_LOG}                                                     |"
wl "|-------------------------------------------------------------------------|"
wl "| ABOUT TO EXIT SCRIPT.                                                   |"
wl "+-------------------------------------------------------------------------+"
wl " "

if [[ $ERRORS = "YES" ]]; then
    exitFailed "FAILED" $UNIQUE_SCRIPT_IDENTIFIER
else
    exitSuccess "SUCCESSFUL" $UNIQUE_SCRIPT_IDENTIFIER
fi
```

## Sub Scripts

sample configuration_setup file

``` wp-block-code
# +----------------------------------------------------------------------------+
# |                          Jeffrey M. Hunter                                 |
# |                      jhunter@idevelopment.info                             |
# |                         www.idevelopment.info                              |
# |----------------------------------------------------------------------------|
# |      Copyright (c) 1998-2009 Jeffrey M. Hunter. All rights reserved.       |
# |----------------------------------------------------------------------------|
# | DATABASE : Oracle                                                          |
# | FILE     : rman_configuration_setup_9i.rcv                                 |
# | CLASS    : Recovery Manager                                                |
# | PURPOSE  : Provides an example RMAN script used to persist Oracle9i RMAN   |
# |            configuration parameters to the control file of the target      |
# |            database. These parameters are used to control default RMAN     |
# |            operations. Some of the operations that can be performed are:   |
# |                                                                            |
# |              - The required number of backups of each datafile.            |
# |              - The number of server processes that will perform Backup     |
# |                and Restore tasks in parallel.                              |
# |              - The directory where on-disk backups will be stored.         |
# |                                                                            |
# |            This script will can be run by RMAN to configure (persist) the  |
# |            default parameters for the target database. For parameters that |
# |            should maintain their default setting, should use the "CLEAR"   |
# |            command to reset the parameter explicitly.                      |
# |                                                                            |
# | NOTE     : As with any code, ensure to test this script in a development   |
# |            environment before attempting to run it in production.          |
# +----------------------------------------------------------------------------+

# +----------------------------------------------------------------------------+
# |                  <<<    RMAN DEFAULT SETTING    >>>                        |
# |----------------------------------------------------------------------------+--------------------------+
# | CONFIGURE RETENTION POLICY TO REDUNDANCY 1; # default                                                 |
# | CONFIGURE BACKUP OPTIMIZATION OFF; # default                                                          |
# | CONFIGURE DEFAULT DEVICE TYPE TO DISK; # default                                                      |
# | CONFIGURE CONTROLFILE AUTOBACKUP OFF; # default                                                       |
# | CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default                       |
# | CONFIGURE DEVICE TYPE DISK PARALLELISM 1; # default                                                   |
# | CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default                                 |
# | CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default                               |
# | CONFIGURE MAXSETSIZE TO UNLIMITED; # default                                                          |
# | CONFIGURE SNAPSHOT CONTROLFILE NAME TO '/u01/app/oracle/product/9.2.0/dbs/snapcf_ORA920.f'; # default |
# +-------------------------------------------------------------------------------------------------------+

# +----------------------------------------------------------------------------+
# | RETENTION POLICY                                                           |
# |----------------------------------------------------------------------------|
# | Used to control how long RMAN will keep backups. This can be configured by |
# | the "number" of backups taken, or by the numbers of "days" to keep. Here   |
# | is an example of both:                                                     |
# |                                                                            |
# |   CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 3 DAYS;                 |
# |                                                                            |
# |   CONFIGURE RETENTION POLICY TO REDUNDANCY 2;                              |
# |                                                                            |
# | Note that when configuring a retention policy, RMAN will *not* cause       |
# | backups to be automatically deleted. The retention policy will, however,   |
# | mark backups as OBSOLETE that have fallen outside the retention period.    |
# | RMAN commands like "REPORT OBSOLETE" and "DELETE OBSOLETE" will work with  |
# | these obsolete backups.                                                    |
# +----------------------------------------------------------------------------+

CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 8 DAYS;

# +----------------------------------------------------------------------------+
# | BACKUP OPTIMIZATION                                                        |
# |----------------------------------------------------------------------------|
# | Used to enable or disable backup optimization. Backup file optimization    |
# | can assist in reducing the space used to backup the database. When         |
# | performing an RMAN backup, a check is done on the file before backup to    |
# | see if the file already exists in a backupset with the same information:   |
# | (dbid, checkpoint, and resetlogs data).                                    |
# |                                                                            |
# | For archive logs, the same file means the same dbid, thread, sequence, and |
# | resetlogs data. If the DELETE INPUT option is used, RMAN deletes all the   |
# | files that would have been backed up, even when the file is not included   |
# | due to file optimization.                                                  |
# |                                                                            |
# | The two possible values for backup optimization is ON and OFF as shown in  |
# | the following example syntax:                                              |
# |                                                                            |
# |   CONFIGURE BACKUP OPTIMIZATION OFF;                                       |
# |                                                                            |
# |   CONFIGURE BACKUP OPTIMIZATION ON;                                        |
# |                                                                            |
# | Use caution when enabling backup optimization if you use a media manager   |
# | that has an expiration policy. The media manager can expire tapes          |
# | containing backups (using its media control software), and RMAN will *not* |
# | make new backups because of optimization. One way to protect from this is  |
# | to run CROSSCHECK periodically to synchronize the repository with the      |
# | media manager.                                                             |
# |                                                                            |
# | Also note that you should consider how backup optimization works with      |
# | regards to the RETENTION POLICY. RMAN will only work with files within     |
# | the retention period. For example, consider performing a backup with       |
# | optimization enabled and a retention period of 3 days. RMAN will only      |
# | compare the datafile with backup sets within the 3 day period - even if    |
# | the datafile it is about to backup hasn't changed within that 3 days.      |
# +----------------------------------------------------------------------------+

CONFIGURE BACKUP OPTIMIZATION CLEAR;

# +----------------------------------------------------------------------------+
# | DEFAULT DEVICE TYPE                                                        |
# |----------------------------------------------------------------------------|
# | Configures the default backup / restore device type for automatic          |
# | channels. The two values for now are TAPE (SBT) and DISK (DISK) - the      |
# | default being DISK. By default, the BACKUP and COPY commands only allocate |
# | channels of the default device type. For example, if you configure         |
# | automatic channels for DISK and sbt and set the default device type to     |
# | DISK, then RMAN only allocates disk channels when you run the BACKUP       |
# | DATABASE command. You can override this behavior either by manually        |
# | allocating channels in a RUN command, or by specifying DEVICE TYPE on the  |
# | BACKUP command itself. The RESTORE command allocates automatic channels of |
# | all configured device types, regardless of the default device type. The    |
# | RESTORE command obeys the PARALLELISM setting for each configured device   |
# | type.                                                                      |
# |                                                                            |
# | The following two examples show the syntax for configuring the default     |
# | device type to TAPE (SBT) - then another command to re-configure the       |
# | default device type to DISK:                                               |
# |                                                                            |
# |   CONFIGURE DEFAULT DEVICE TYPE TO SBT;                                    |
# |                                                                            |
# |   CONFIGURE DEFAULT DEVICE TYPE TO DISK;                                   |
# |                                                                            |
# +----------------------------------------------------------------------------+

CONFIGURE DEFAULT DEVICE TYPE TO DISK;

# +----------------------------------------------------------------------------+
# | CONTROLFILE AUTOBACKUP                                                     |
# |----------------------------------------------------------------------------|
# | Starting with Oracle9i, RMAN offers the ability to backup the control file |
# | and the database parameter file (SPFILE only) and have this take place     |
# | automatically by default after each backup. By default, this feature is    |
# | not enabled. This feature can be enabled or disabled using the example     |
# | syntax below:                                                              |
# |                                                                            |
# |   CONFIGURE CONTROLFILE AUTOBACKUP ON;                                     |
# |                                                                            |
# |   CONFIGURE CONTROLFILE AUTOBACKUP OFF;                                    |
# |                                                                            |
# | When the CONTROLFILE AUTOBACKUP feature is enabled, then RMAN performs a   |
# | control file autobackup in the following circumstances:                    |
# |                                                                            |
# |     - After every BACKUP or COPY command issued at the RMAN prompt.        |
# |     - Whenever a BACKUP or COPY command within a RUN block is followed     |
# |       by a command that is neither BACKUP nor COPY.                        |
# |     - At the end of every RUN block if the last command in the block was   |
# |       either BACKUP or COPY.                                               |
# |     - After database structural changes such as adding a new tablespace,   |
# |       altering the state of a tablespace or datafile (for example,         |
# |       bringing it online), adding a new online redo log, renaming a file,  |
# |       adding a new redo thread, and so forth. This type of autobackup,     |
# |       unlike autobackups that occur in the preceding circumstances, goes   |
# |       only to disk. You can run:                                           |
# |       CONFIGURE CONTROLFILE AUTOBACKUP FOR DEVICE TYPE DISK                |
# |       to set a nondefault disk location.                                   |
# |                                                                            |
# | The "first channel" allocated during the BACKUP or COPY job creates the    |
# | autobackup and places it "into its own backup set"; for post-structural    |
# | autobackups, the "default disk channel" makes the backup.                  |
# |                                                                            |
# | RMAN writes both the CONTROLFILE and the SPFILE (if the database was       |
# | started with an SPFILE) to the same backup piece.                          |
# | After the CONTROLFILE AUTOBACKUP completes, Oracle writes a message        |
# | containing the complete path of the backup piece and the device type to    |
# | the alert log.                                                             |
# |                                                                            |
# | RMAN automatically backs up the current control file using the default     |
# | format of %F (see entry for CONFIGURE CONTROLFILE AUTOBACKUP FORMAT for an |
# | explanation of this substitution variable). You can change this format     |
# | using the CONFIGURE CONTROLFILE AUTOBACKUP FORMAT and SET CONTROLFILE      |
# | AUTOBACKUP FORMAT commands.                                                |
# |                                                                            |
# | When this feature is disabled, any BACKUP command that includes datafile 1 |
# | (including BACKUP DATABASE) automatically includes the current control     |
# | file and server parameter file in the backup set. Otherwise, RMAN does not |
# | include these files.                                                       |
# |                                                                            |
# | NOTE: You may see occasions where RMAN will sometimes automatically        |
# | include the CONTROLFILE and SPFILE in the backupsets during a backup. This |
# | is normal and is documented (above). When CONTROLFILE AUTOBACKUP is not    |
# | enabled, the CONTROLFILE and SPFILE, (if the database was started with an  |
# | spfile), are included automatically when datafile 1 is backed up. Consider |
# | the  following backup:                                                     |
# |                                                                            |
# |     RMAN> backup datafile 1;                                               |
# |                                                                            |
# | It will include the CONTROLFILE and the SPFILE (when the database is       |
# | started with a spfile).                                                    |
# +----------------------------------------------------------------------------+

CONFIGURE CONTROLFILE AUTOBACKUP ON;

# +----------------------------------------------------------------------------+
# | CONTROLFILE AUTOBACKUP FORMAT                                              |
# |----------------------------------------------------------------------------|
# | Configures the default filename format for the control file autobackup on  |
# | the specified device type. By default, the initial format is %F for all    |
# | devices. Any default format string specified with CONFIGURE must include   |
# | the %F substitution variable (see BACKUP). This variable translates into   |
# | c-IIIIIIIIII-YYYYMMDD-QQ, where:                                           |
# |                                                                            |
# |   - IIIIIIIIII  stands for the DBID. (The DBID is printed in decimal so    |
# |                 that it can be easily associated with the target database. |
# |   - YYYYMMDD    is a time stamp in the Gregorian calendar of the day the   |
# |                 backup is generated.                                       |
# |   - QQ          is the sequence in hexadecimal number that starts with 00  |
# |                 and has a maximum of 'FF' (256).                           |
# |                                                                            |
# | Specify CLEAR to return the format to the default %F.                      |
# |                                                                            |
# | NOTE: The %F tag is essential for RMAN to be able to restore the file      |
# | without a recovery catalog.                                                |
# +----------------------------------------------------------------------------+

CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/backup/${ORACLE_SID}/inc0/%F';

# +----------------------------------------------------------------------------+
# | PARALLELISM                                                                |
# |----------------------------------------------------------------------------|
# | Configure RMAN to use [n] number of disk channels for backup, restore,     |
# | recovery, and maintenance operations. Device types that are eligible for   |
# | use in jobs that use automatic channels can sets the degree of channel     |
# | parallelism. (The DISK device type is the default)                         |
# |                                                                            |
# | The PARALLELISM parameter specifies the number of automatic channels of    |
# | the specified device type allocated for RMAN jobs. RMAN always allocates   |
# | the number of channels specified by PARALLELISM, although it may actually  |
# | use only a subset of these channels.                                       |
# |                                                                            |
# | By default, PARALLELISM = 1.                                               |
# |                                                                            |
# | Take the following example; you can set PARALLELISM for DISK backups to 3. |
# | If you configure automatic channels of type disk and tape, and set the     |
# | default device type as disk, then RMAN allocates three disk channels when  |
# | you run BACKUP DATABASE at the RMAN prompt.                                |
# |                                                                            |
# | To change the parallelism for a device type to [n], run:                   |
# |                                                                            |
# |     CONFIGURE DEVICE TYPE [DISK | SBT]  PARALLELISM [n];                   |
# |                                                                            |
# | Here are several examples of how to configure PARALLELISM to 3 for sbt and |
# | then change it to 2:                                                       |
# |                                                                            |
# |     CONFIGURE DEVICE TYPE sbt PARALLELISM 3;                               |
# |     CONFIGURE DEVICE TYPE sbt PARALLELISM 2;                               |
# |                                                                            |
# | Another example that configures parallelism to 2 for automatic disk        |
# | channel.                                                                   |
# |                                                                            |
# |     CONFIGURE DEVICE TYPE DISK PARALLELISM 2;                              |
# |                                                                            |
# | NOTE: If you configure [n] manually numbered channels, the PARALLELISM     |
# | setting can be greater than or less than [n]. For example, you can         |
# | manually number 10 automatic channels and configure PARALLELISM to 2 or    |
# | even 12.                                                                   |
# +----------------------------------------------------------------------------+

CONFIGURE DEVICE TYPE DISK CLEAR;

# +----------------------------------------------------------------------------+
# | DATABASE BACKUP COPIES                                                     |
# | ARCHIVELOG BACKUP COPIES                                                   |
# |----------------------------------------------------------------------------|
# | Specifies the number of copies of each backup set for DATAFILE (both       |
# | datafiles and control files) or ARCHIVELOG files on the specified device   |
# | type, from 1 (default) to 4. If duplexing is specified in the BACKUP       |
# | command or in a SET BACKUP COPIES command, then the CONFIGURE setting is   |
# | overridden.                                                                |
# |                                                                            |
# | Here is the general syntax:                                                |
# |                                                                            |
# |     {ARCHIVELOG| DATAFILE}                                                 |
# |         BACKUP COPIES FOR DEVICE                                           |
# |         TYPE deviceSpecifier                                               |
# |         TO integer;                                                        |
# |                                                                            |
# | NOTE: Control file autobackups on disk are a special case and are never    |
# |       duplexed. RMAN always writes one and only copy.                      |
# +----------------------------------------------------------------------------+

CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK CLEAR;
CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK CLEAR;

# +----------------------------------------------------------------------------+
# | CHANNEL CONFIGURATION                                                      |
# |----------------------------------------------------------------------------|
# | Specifies the standard or AUXILIARY channel that you are configuring or    |
# | clearing, as well as the device type (DISK or sbt) of the channel. You can |
# | either configure a "generic channel" or specify a channel by number, where |
# | "integer" is less than 255.                                                |
# |                                                                            |
# | Here is the general syntax:                                                |
# |                                                                            |
# |     CHANNEL integer DEVICE TYPE deviceSpecifier allocOperandList;          |
# |                                                                            |
# |         integer              You can specify a channel by number. You may  |
# |                              also exclude an integer, in which case you    |
# |                              are specifying a general channel.             |
# |         deviceSpecifier      {DISK | SBT}                                  |
# |         allocOperandList     /* Specifies control options for the  */      |
# |                              /* allocated channel                  */      |
# |                              [PARMS         = 'channel_parms']             |
# |                              [CONNECT       = (quoted string)]             |
# |                              [FORMAT        = 'format_sring']              |
# |                              [MAXPIECESIZE  = integer [K|M|G]]             |
# |                              [RATE          = integer [K|M|G]]             |
# |                              [MAXOPENFILES  = integer]                     |
# |                              [SEND          = 'command']                   |
# |                                                                            |
# | NOTE: The RATE parameter is intended to slow down a backup so that you can |
# |       run it in the background with as little effect as possible on OLTP   |
# |       operations. The RATE parameter specifies units of bytes/second. Test |
# |       to find a value that improves performance of your queries while      |
# |       still letting RMAN complete the backup in a reasonable amount of     |
# |       time. Note that RATE is not designed to increase backup throughput,  |
# |       but to decrease backup throughput so that more disk bandwidth is     |
# |       available for other database operations.                             |
# |                                                                            |
# | NOTE: The CONNECT parameter Specifies a connect string to the database     |
# |       instance where RMAN should conduct the backup or restore operations. |
# |       Use this parameter to spread the work of backup or restore           |
# |       operations across different instances in an Oracle Real Application  |
# |       Clusters configuration. If you do not specify this parameter, and if |
# |       you did not specify the AUXILIARY option, then RMAN conducts all     |
# |       operations on the target database instance specified by the          |
# |       command-line parameter or the instance connected to when you issued  |
# |       the CONNECT command. Typically, you should not use the CONNECT       |
# |       parameter in conjunction with the AUXILIARY option.                  |
# |                                                                            |
# | I generally use this parameter to configure the location (directory)       |
# | and file format where RMAN will write disk backups to. Here are some of    |
# | the substitution variables that can be used in the backup set file name:   |
# |                                                                            |
# |     %d      Database name                                                  |
# |     %s      Backup set number                                              |
# |     %p      Backup piece number                                            |
# |     %t      4-byte timestamp                                               |
# |                                                                            |
# | If you configure a generic channel (that is, if you do not specify a       |
# | channel number), then RMAN uses the generic settings for every             |
# | parallelized channel except any channel number that you have explicitly    |
# | configured. In other words, a generic channel setting specifies options    |
# | for all channels not configured explicitly.                                |
# |                                                                            |
# | NOTE: For generic channels of a specified device type, a new command       |
# |       erases previous settings for this device type. Assume that you run   |
# |       these commands:                                                      |
# |                                                                            |
# |       CONFIGURE CHANNEL DEVICE TYPE sbt MAXPIECESIZE 1G;                   |
# |       CONFIGURE CHANNEL DEVICE TYPE sbt RATE 1700K;                        |
# |                                                                            |
# |       The second command erases the MAXPIECESIZE setting of the first      |
# |       command.                                                             |
# |                                                                            |
# | If AUXILIARY is specified, then this configuration is used only for        |
# | channels allocated at the auxiliary instance. If no auxiliary device       |
# | configuration is specified, and if RMAN needs to automatically allocate    |
# | auxiliary channels, then RMAN uses the target database device              |
# | configuration. It is not necessary to specify configuration information    |
# | for auxiliary channels unless they require different parameters from the   |
# | target channels.                                                           |
# |                                                                            |
# | NOTE: It is useful to understand how the CLEAR command works for this      |
# |       parameter. CLEAR Clears the specified channel. For example,          |
# |       CONFIGURE CHANNEL 1 DEVICE TYPE DISK CLEAR returns only channel 1 to |
# |       its default, whereas CONFIGURE CHANNEL DEVICE TYPE DISK CLEAR        |
# |       returns the generic disk channel to its default. Note that you       |
# |       cannot specify any other channel options (for example, PARMS) when   |
# |       you specify CLEAR.                                                   |
# +----------------------------------------------------------------------------+
CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO BACKUPSET;

CONFIGURE CHANNEL 1  DEVICE TYPE DISK  FORMAT '/backup/${ORACLE_SID}/inc0/backup_db_%d_S_%s_P_%p_T_%t' MAXPIECESIZE 1024m;
CONFIGURE CHANNEL 2  DEVICE TYPE DISK  FORMAT '/backup/${ORACLE_SID}/inc0/backup_db_%d_S_%s_P_%p_T_%t' MAXPIECESIZE 1024m;
CONFIGURE CHANNEL 3  DEVICE TYPE DISK  FORMAT '/backup/${ORACLE_SID}/inc0/backup_db_%d_S_%s_P_%p_T_%t' MAXPIECESIZE 1024m;
CONFIGURE CHANNEL 4  DEVICE TYPE DISK  FORMAT '/backup/${ORACLE_SID}/inc0/backup_db_%d_S_%s_P_%p_T_%t' MAXPIECESIZE 1024m;

# +----------------------------------------------------------------------------+
# | MAXSETSIZE                                                                 |
# |----------------------------------------------------------------------------|
# | Specifies the maximum size of each backup set created on a channel. By     |
# | default MAXSETSIZE is set to UNLIMITED, meaning that it is disabled.       |
# |                                                                            |
# | Here is the general syntax:                                                |
# |                                                                            |
# |     CONFIGURE MAXSETSIZE TO { integer [K|M|G] | UNLIMITED | CLEAR };       |
# |                                                                            |
# | TO integer       Specifies the maximum set size in bytes, kilobytes (K),   |
# |                  megabytes (M), or gigabtyes (G). The default setting is   |
# |                  in bytes and is rounded down to kilobtyes. For example,   |
# |                  if you set MAXSETSIZE to 5000, RMAN sets the maximum set  |
# |                  size at 4 kilobytes (that is, 4096 bytes), which is the   |
# |                  lower kilobyte boundary of 5000. The minimum value must   |
# |                  be greater than or equal to the database block size.      |
# | TO UNLIMITED     Specifies that there is no size limit for backup sets.    |
# | CLEAR            Resets the maximum set size to its default value          |
# |                  (UNLIMITED).                                              |
# +----------------------------------------------------------------------------+

CONFIGURE MAXSETSIZE CLEAR;

# +----------------------------------------------------------------------------+
# | SNAPSHOT CONTROLFILE                                                       |
# |----------------------------------------------------------------------------|
# | Configures the snapshot control file filename to 'filename'. If you run    |
# | CONFIGURE SNAPSHOT CONTROLFILE NAME CLEAR, then RMAN sets the snapshot     |
# | control file name to its default.                                          |
# |                                                                            |
# | The default value for the snapshot control file name is platform-specific  |
# | and dependent on the Oracle home. For example, the default on some UNIX    |
# | system is ?/dbs/snapcf_@.f. If you clear the control file name, and you    |
# | change the Oracle home, then the default location of the snapshot control  |
# | file changes as well.                                                      |
# |                                                                            |
# |     SNAPSHOT CONTROLFILE NAME TO 'filename';                               |
# +----------------------------------------------------------------------------+

CONFIGURE SNAPSHOT CONTROLFILE NAME CLEAR;

# +----------------------------------------------------------------------------+
# | EXCLUDE TABLESPACE                                                         |
# |----------------------------------------------------------------------------|
# | Excludes the specified tablespace from BACKUP DATABASE commands. Note that |
# | you cannot exclude the SYSTEM tablespace. By default, each tablespace is   |
# | not excluded, that is, the exclude functionality is disabled.              |
# |                                                                            |
# | The exclusion is stored as an attribute of the tablespace, not the         |
# | individual datafiles, so the exclusion applies to any files that are added |
# | to this tablespace in the future.                                          |
# |                                                                            |
# | If you run CONFIGURE ... CLEAR on a tablespace after excluding it, then it |
# | returns to the default configuration of "not excluded."                    |
# |                                                                            |
# | You can still back up the configured tablespace by explicitly specifying   |
# | it in a BACKUP command or by specifying the NOEXCLUDE option on a BACKUP   |
# | DATABASE command.                                                          |
# |                                                                            |
# | The general syntax is:                                                     |
# |                                                                            |
# |     EXCLUDE FOR TABLESPACE tablespace_name;                                |
# |                                                                            |
# +----------------------------------------------------------------------------+

# CONFIGURE EXCLUDE FOR TABLESPACE USERS_READ_TBS;
```
