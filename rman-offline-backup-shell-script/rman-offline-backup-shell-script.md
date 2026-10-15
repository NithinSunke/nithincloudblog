---
title: "rman offline backup shell script"
author: "Sunke Nithin"
date: 2019-07-03
categories: ["SHELL SCRIPTS"]
tags: []
source: https://nithinsunke.wordpress.com/2019/07/03/rman-offline-backup-shell-script/
---

# rman offline backup shell script

##!/bin/sh
    # ----------------------------
    # ORGANIZATION INFORMATION
    # ----------------------------
    ORGANIZATION_NAME="sunkeTech.info"
    # ----------------------------
    # SCRIPT PARAMETER VARIABLES
    # ----------------------------
    TARGET_DB_NAME=sunkdev
    TARGET_DBA_USERNAME=sys
    TARGET_DBA_PASSWORD='<sys_password>'
    RMAN_RECOVERY_CATALOG=CATALOG
    CATALOG_DB_NAME=rmandv
    CATALOG_DBA_USERNAME=rmanus
    CATALOG_DBA_PASSWORD='<catalog_password>'
    BACKUP_PROFILE=1

    # ----------------------------

    # ORACLE ENVIRONMENT VARIABLES

    # ----------------------------

    ORACLE_BASE=/a01/oracle/11.2.0/db

    ORACLE_ADMIN_DIR=${ORACLE_BASE}/admin

    ORACLE_DIAG_DIR=${ORACLE_BASE}/diag

    # ----------------------------

    # CUSTOM DIRECTORIES

    # ----------------------------

    CUSTOM_ORACLE_DIR=/d01/scripts

    CUSTOM_ORACLE_SHELL_DIR=${CUSTOM_ORACLE_DIR}/shell

    CUSTOM_ORACLE_LOG_DIR=${CUSTOM_ORACLE_DIR}/log

    CUSTOM_ORACLE_CMD_DIR=${CUSTOM_ORACLE_DIR}/cmd

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

    # ----------------------------

    # EMAIL PREFERENCES

    # ----------------------------

    # LIST ALL ADMINISTRATIVE

    # EMAIL ADDRESSES WHO WILL BE

    # RESPONSIBLE FOR MONITORING

    # AND RECEIVING EMAIL FROM

    # THIS SCRIPT.

    # ----------------------------

    # THREE EMAIL RECIPIENT LISTS

    # EXIST:

    # 1) WHEN THIS SCRIPT CALLS

    # exitSuccess()

    # 2) WHEN THIS SCRIPT CALLS

    # exitFailed()

    # ----------------------------

    # MULTIPLE EMAIL ADDRESSES

    # SHOULD ALL BE LISTED IN

    # DOUBLE-QUOTES SEPARATED BY A

    # SINGLE SPACE.

    # ----------------------------

    MAIL_RECIPIENT_LIST_EXIT_SUCCESS="dba@example.com"

    MAIL_RECIPIENT_LIST_EXIT_FAILED="dba@example.com"

    MAIL_FROM="${ORGANIZATION_NAME} Database Support <dba@example.com>"

    MAIL_REPLYTO="${ORGANIZATION_NAME} Database Support <dba@example.com>"

    MAIL_TO_NAME="${ORGANIZATION_NAME} Database Support"

    MAIL_TEMP_FILE_NAME=${CUSTOM_ORACLE_TEMP_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}.mhr

    # +------------------------------------------------------------------------+

    # | |

    # | DEFINE ALL INTERNAL GLOBAL VARIABLES |

    # | |

    # +------------------------------------------------------------------------+

    # ----------------------------

    # DATE VARIABLES

    # ----------------------------

    START_DATE=`date`

    START_DATE_LOG=`date +"%Y%m%d_%H%M%S"`

    START_DATE_PRINT=`date +"%m/%d/%Y %r %Z"`

    # ----------------------------

    # MISCELLANEOUS VARIABLES

    # ----------------------------

    HOST_RVAL_SUCCESS=0

    HOST_RVAL_FAILED=1

    HIDE_PASSWORD_STRING="xxxxxxxxxxxxx"

    # ----------------------------

    # LOG AND TEMP FILE VARIABLES

    # ----------------------------

    LOG_FILE_NAME=${CUSTOM_ORACLE_LOG_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}_${START_DATE_LOG}.log

    LOG_FILE_NAME_NODATE=${CUSTOM_ORACLE_LOG_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}.log

    CHECK_SCRIPT_RUNNING_FLAG_FILE=${CUSTOM_ORACLE_TEMP_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}.running

    SQL_OUTPUT_TEMP_FILE_NAME=${CUSTOM_ORACLE_TEMP_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}.lst

    # +------------------------------------------------------------------------+

    # | |

    # | DEFINE ALL INTERNAL GLOBAL FUNCTIONS |

    # | |

    # +-----------------------------------------------------------------------+

    function wl {

    echo "${1}" >> ${LOG_FILE_NAME}

    echo "${1}"

    }

    function printScriptParameterVariables {

    wl " "

    wl "================================================================"

    wl " PRINT SCRIPT PARAMETER VARIABLES "

    wl "================================================================"

    wl " "

    wl "TRACE> TARGET_DB_NAME set to $TARGET_DB_NAME"

    wl "TRACE> TARGET_SID set to $TARGET_SID"

    wl "TRACE> TARGET_DBA_USERNAME set to $TARGET_DBA_USERNAME"

    wl "TRACE> TARGET_DBA_PASSWORD set to $HIDE_PASSWORD_STRING"

    wl "TRACE> RMAN_RECOVERY_CATALOG set to $RMAN_RECOVERY_CATALOG"

    if [[ ! -z $RMAN_RECOVERY_CATALOG ]]; then

    wl "TRACE> CATALOG_DB_NAME set to $CATALOG_DB_NAME"

    wl "TRACE> CATALOG_DBA_USERNAME set to $CATALOG_DBA_USERNAME"

    wl "TRACE> CATALOG_DBA_PASSWORD set to $HIDE_PASSWORD_STRING"

    else

    wl "TRACE> CATALOG_DBA_PASSWORD set to null"

    fi

    }

    function showUsage {

    wl "Usage: ${SCRIPT_NAME}"

    wl " "

    }

    function startLogging {

    wl "+=======================================================================+"

    wl "| |"

    wl "| START TIME |"

    wl "| |"

    wl "| $START_DATE |"

    wl "| |"

    wl "+=======================================================================+"

    return

    }

    function stopLogging {

    END_DATE=`date`

    wl " "

    wl "+=========================================================================+"

    wl "| |"

    wl "| FINISH TIME |"

    wl "| |"

    wl "| $END_DATE |"

    wl "| |"

    wl "+=========================================================================+"

    return

    }

    function initializeScript {

    # ----------------------------------------

    # INITIALIZE LOG FILE

    # ----------------------------------------

    startLogging

    # ----------------------------------------

    # PRINT SCRIPT PARAMETER VARIABLES

    # ----------------------------------------

    printScriptParameterVariables

    return

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

    function sendEmail {

    # -------------------------------

    # POSSIBLE L_SEVERITY VALUES ARE:

    # SUCCESSFUL

    # RUNNING

    # FAILED

    # -------------------------------

    typeset -r L_SEVERITY=${1}

    typeset -r L_EMAIL_ADDRESS_LIST=${2}

    typeset L_IMPORTANCE

    typeset L_X_PRIORITY

    typeset L_X_MSMAIL_PRIORITY

    typeset L_EMAIL_ADDRESS

    case ${L_SEVERITY} in

    SUCCESSFUL)

    L_IMPORTANCE="Normal"

    L_X_PRIORITY="3"

    L_X_MSMAIL_PRIORITY="Normal"

    ;;

    RUNNING)

    L_IMPORTANCE="High"

    L_X_PRIORITY="1"

    L_X_MSMAIL_PRIORITY="High"

    ;;

    WARNING)

    L_IMPORTANCE="High"

    L_X_PRIORITY="1"

    L_X_MSMAIL_PRIORITY="High"

    ;;

    FAILED)

    L_IMPORTANCE="High"

    L_X_PRIORITY="1"

    L_X_MSMAIL_PRIORITY="High"

    ;;

    *)

    L_IMPORTANCE="High"

    L_X_PRIORITY="1"

    L_X_MSMAIL_PRIORITY="High"

    ;;

    esac

    wl " "

    wl "TRACE> Emailing the following recipients:"

    wl " "

    for L_EMAIL_ADDRESS in $L_EMAIL_ADDRESS_LIST; do

    wl " $L_EMAIL_ADDRESS"

    done

    wl " "

    for L_EMAIL_ADDRESS in $L_EMAIL_ADDRESS_LIST; do

    {

    #echo "Subject: [${HOSTNAME_SHORT_UPPER}] - ${L_SEVERITY}: ${SCRIPT_NAME} "

    SUBJECT="[${HOSTNAME_SHORT_UPPER}] - ${L_SEVERITY}_${SCRIPT_NAME}"

    echo $SUBJECT

    #echo "To: ${MAIL_TO_NAME} <${L_EMAIL_ADDRESS}>"

    TOMAIL="${MAIL_TO_NAME} <${L_EMAIL_ADDRESS}>"

    echo $TOMAIL

    #echo "From: ${MAIL_FROM}"

    FROMMAIL="${MAIL_FROM}"

    echo $FROMMAIL

    #echo "Reply-To: ${MAIL_REPLYTO}"

    echo ""

    cat ${LOG_FILE_NAME}

    } > ${MAIL_TEMP_FILE_NAME}

    # sendmail -v $L_EMAIL_ADDRESS < ${MAIL_TEMP_FILE_NAME} | tee -a $LOG_FILE_NAME

    echo "Message Body Here" | mailx -s "${SUBJECT}" -r "${FROMMAIL}" -a $LOG_FILE_NAME "${L_EMAIL_ADDRESS}"

    wl "TRACE> Sent email to $L_EMAIL_ADDRESS"

    rm -f $MAIL_TEMP_FILE_NAME | tee -a $LOG_FILE_NAME

    done

    return

    }

    function exitSuccess {

    wl " "

    wl "TRACE> +----------------------------------------------+"

    wl "TRACE> | SUCCESSFUL |"

    wl "TRACE> +----------------------------------------------+"

    wl " "

    wl "TRACE> Exiting script (${HOST_RVAL_SUCCESS})."

    wl " "

    removeScriptRunFlagFile

    stopLogging

    backupCurrentLogFile

    sendEmail "SUCCESSFUL" "${MAIL_RECIPIENT_LIST_EXIT_SUCCESS}"

    exit ${HOST_RVAL_SUCCESS}

    }

    function exitFailed {

    typeset -r L_SEVERITY=$1

    wl " "

    wl "TRACE> +----------------------------------------------+"

    wl "TRACE> | !!!!!!!! CRITICAL ERROR !!!!!!!! |"

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

    sendEmail "FAILED" "${MAIL_RECIPIENT_LIST_EXIT_FAILED}"

    exit ${HOST_RVAL_FAILED}

    }

    function checkScriptAlreadyRunning {

    typeset -r L_SCRIPT_NAME=${SCRIPT_NAME}

    wl " "

    wl "TRACE> Check that this script (${L_SCRIPT_NAME}) is not already running on this host."

    wl " "

    wl "TRACE> Looking for script run flag file (${CHECK_SCRIPT_RUNNING_FLAG_FILE})."

    if [ -f $CHECK_SCRIPT_RUNNING_FLAG_FILE ]; then

    wl " "

    wl "TRACE> WARNING: Found ${L_SCRIPT_NAME} already running on this host. Exiting script."

    exitFailed "RUNNING"

    else

    wl " "

    wl "TRACE> Did not find this script (${L_SCRIPT_NAME}) already running on this host. Setting run flag and continuing script..."

    touch $CHECK_SCRIPT_RUNNING_FLAG_FILE

    fi

    wl " "

    return

    }

    function verifyOSUserLogin {

    typeset -r L_CHECK_USER_NAME=oracle

    typeset -r L_REQUIRED=TRUE

    typeset L_UID

    # L_UID=`/usr/bin/id|awk -F( '{print $2}'|awk -F) '{print $1}'`

    L_UID=`id|awk -F( '{print $2}'|awk -F) '{print $1}'`

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

    typeset -r L_SID_NAME=sunkdev

    typeset L_COMMAND

    wl " "

    wl "TRACE> Check that the Oracle instance (${L_SID_NAME}) is up."

    L_COMMAND="ps -ef | grep "ora_smon_$L_SID_NAME$" | grep -v 'grep'"

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

    exitFailed "FAILED"

    fi

    return

    }

    function verifyTNSConnectString {

    typeset -r L_DB_NAME=${1}

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

    exitFailed "FAILED"

    fi

    return

    }

    function verifyDatabaseLoginCredentials {

    typeset -r L_DB_NAME=${1}

    typeset -r L_DBA_USERNAME=${2}

    typeset -r L_DBA_PASSWORD=${3}

    typeset -r L_SYSDBA_PRIVS=${4}

    typeset L_SYSDBA_PRIVS_TXT

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

    exitFailed "FAILED"

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

    # | |

    # | DEFINE ALL CUSTOM GLOBAL FUNCTIONS |

    # | |

    # +----------------------------------------------------------------------------+

    function RMANBackup {

    typeset -r L_TARGET_DB_NAME=${TARGET_DB_NAME}

    typeset -r L_TARGET_DBA_USERNAME=${TARGET_DBA_USERNAME}

    typeset -r L_TARGET_DBA_PASSWORD=${TARGET_DBA_PASSWORD}

    typeset -r L_CATALOG_DB_NAME=${CATALOG_DB_NAME}

    typeset -r L_CATALOG_DBA_USERNAME=${CATALOG_DBA_USERNAME}

    typeset -r L_CATALOG_DBA_PASSWORD=${CATALOG_DBA_PASSWORD}

    typeset L_RMAN_CONNECT_TARGET

    typeset L_RMAN_CONNECT_CATALOG

    wl " "

    wl "TRACE> +---------------------------------+"

    wl "TRACE> | RMAN Backup Profile Description |"

    wl "TRACE> |------------------------------------------------------------------------+"

    wl "TRACE> | * Full database backup + archivelog delete input to disk as BACKUPSET. |"

    wl "TRACE> | * Primary database if using Data Guard. |"

    wl "TRACE> | * Compatibility: 10g / 11g / 12c |"

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

    # ---------------------------------------------

    # PERFORM FULL RMAN BACKUP

    # ---------------------------------------------

    $ORACLE_HOME/bin/rman <<EOF | tee -a $LOG_FILE_NAME

    $L_RMAN_CONNECT_TARGET

    $L_RMAN_CONNECT_CATALOG

    startup mount

    run {

    allocate channel c1 device type disk;

    allocate channel c2 device type disk;

    allocate channel c3 device type disk;

    report schema;

    crosscheck backup of database;

    crosscheck backup of archivelog all;

    crosscheck backup of controlfile;

    crosscheck backup of spfile;

    crosscheck archivelog all;

    backup as compressed backupset check logical incremental level = 0 database format '/d01/scripts/bkp/backup_%d_%I_set%s_piece%p_copy%c_%T_%U' filesperset 10 TAG=LVL0_FULL_DB plus archivelog not backed up 1 times format '/d01/scripts/bkp/backup_arch_%I_%e_%T_%U' TAG=FULL_ARCH delete input;

    delete noprompt force expired backup;

    delete noprompt force expired archivelog all;

    delete noprompt force expired copy;

    delete noprompt force obsolete;

    delete noprompt force archivelog all completed before 'sysdate-2';

    backup current controlfile format '/d01/scripts/bkp/backup_ctrl_%I_%T_%U';

    backup spfile format '/d01/scripts/bkp/backup_spfile_%I_%T_%U';

    }

    shutdown abort;

    exit;

    EOF

    }

    function startup {

    $ORACLE_HOME/bin/sqlplus /nolog <<eof | tee -a $LOG_FILE_NAME

    conn / as sysdba

    startup

    select name,open_mode from v$database;

    exit

    eof

    }

    function shutdown {

    $ORACLE_HOME/bin/sqlplus /nolog <<eof | tee -a $LOG_FILE_NAME

    conn / as sysdba

    select name,open_mode from v$database;

    shutdown immediate

    exit;

    eof

    }

    # +----------------------------------------------------------------------------+

    # | |

    # | SCRIPT STARTS HERE |

    # | |

    # +----------------------------------------------------------------------------+

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| VERIFY OS LOGIN. |"

    wl "+-------------------------------------------------------------------------+"

    verifyOSUserLogin

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| VERIFY AN INSTANCE OF THIS SCRIPT IS NOT ALREADY RUNNING. |"

    wl "+-------------------------------------------------------------------------+"

    checkScriptAlreadyRunning

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| DISPLAY ALL SCRIPT ENVIRONMENT VARIABLES. |"

    wl "+-------------------------------------------------------------------------+"

    wl " "

    wl "================================================================"

    wl " GLOBAL SCRIPT VARIABLES "

    wl "================================================================"

    wl "ORGANIZATION NAME : $ORGANIZATION_NAME"

    wl "SCRIPT : $SCRIPT_NAME"

    wl "START DATE/TIME : $START_DATE"

    wl "CHECK_SCRIPT_RUNNING_FLAG_FILE : $CHECK_SCRIPT_RUNNING_FLAG_FILE"

    wl "HOST_NAME : $HOSTNAME"

    wl "HOST_NAME (UPPER) : $HOSTNAME_UPPER"

    wl "HOST_NAME (SHORT) : $HOSTNAME_SHORT"

    wl "HOST_NAME (SHORT/UPPER) : $HOSTNAME_SHORT_UPPER"

    wl "ORACLE_BASE : $ORACLE_BASE"

    wl "ORACLE_HOME : $ORACLE_HOME"

    wl "ORACLE_ADMIN_DIR : $ORACLE_ADMIN_DIR"

    wl "ORACLE_DIAG_DIR : $ORACLE_DIAG_DIR"

    wl "LOG_FILE_NAME : $LOG_FILE_NAME"

    wl "LOG_FILE_NAME_NODATE : $LOG_FILE_NAME_NODATE"

    wl "LOG_FILE_ARCHIVE_OBSOLETE_DAYS : $LOG_FILE_ARCHIVE_OBSOLETE_DAYS"

    wl "EMAIL RECIPIENT LIST - EXIT (S) : ... <LIST> ..."

    for email_address in $MAIL_RECIPIENT_LIST_EXIT_SUCCESS; do

    wl " $email_address"

    done

    wl "EMAIL RECIPIENT LIST - EXIT (W) : ... <LIST> ..."

    for email_address in $MAIL_RECIPIENT_LIST_EXIT_WARNING; do

    wl " $email_address"

    done

    wl "EMAIL RECIPIENT LIST - EXIT (F) : ... <LIST> ..."

    for email_address in $MAIL_RECIPIENT_LIST_EXIT_FAILED; do

    wl " $email_address"

    done

    wl " "

    wl "==========================================================="

    wl " COMMAND-LINE ARGUMENTS "

    wl "==========================================================="

    wl "TARGET_DB_NAME (P1) : $TARGET_DB_NAME"

    wl "TARGET_DBA_USERNAME (P3) : $TARGET_DBA_USERNAME"

    wl "TARGET_DBA_PASSWORD (P4) : $HIDE_PASSWORD_STRING"

    wl "RMAN_RECOVERY_CATALOG (P6) : $RMAN_RECOVERY_CATALOG"

    wl "CATALOG_DB_NAME (P7) : $CATALOG_DB_NAME"

    wl "CATALOG_DBA_USERNAME (P8) : $CATALOG_DBA_USERNAME"

    if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then

    wl "CATALOG_DBA_PASSWORD (P9) : $HIDE_PASSWORD_STRING"

    else

    wl "CATALOG_DBA_PASSWORD (P9) : "

    fi

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| THIS SCRIPT IS TO BE RUN FROM THE DATABASE SERVER HOSTING THE TARGET |"

    wl "| DATABASE. VERIFY ORACLE INSTANCE IS UP AND RUNNING ON THIS HOST. |"

    wl "+-------------------------------------------------------------------------+"

    verifyOracleSID

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| VERIFY THE ORACLE TNS CONNECT STRING IS VALID TO THE TARGET DATABASE. |"

    wl "+-------------------------------------------------------------------------+"

    verifyTNSConnectString $TARGET_DB_NAME

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| VERIFY LOG IN CREDENTIALS TO THE TARGET DATABASE. |"

    wl "+-------------------------------------------------------------------------+"

    verifyDatabaseLoginCredentials $TARGET_DB_NAME $TARGET_DBA_USERNAME $TARGET_DBA_PASSWORD "SYSDBA"

    # +----------------------------------------------------------------------------+

    # | |

    # | CUSTOM SCRIPT TASKS (BEGIN) |

    # | |

    # +----------------------------------------------------------------------------+

    if [[ $RMAN_RECOVERY_CATALOG = "CATALOG" ]]; then

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| VERIFY THE ORACLE TNS CONNECT STRING IS VALID TO THE CATALOG DATABASE. |"

    wl "+-------------------------------------------------------------------------+"

    verifyTNSConnectString $CATALOG_DB_NAME

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| VERIFY LOG IN CREDENTIALS TO THE CATALOG DATABASE. |"

    wl "+-------------------------------------------------------------------------+"

    verifyDatabaseLoginCredentials $CATALOG_DB_NAME $CATALOG_DBA_USERNAME $CATALOG_DBA_PASSWORD

    fi

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| PERFORM RMAN DATABASE OFFLINE BACKUP. |"

    wl "+-------------------------------------------------------------------------+"

    wl " "

    wl "TRACE> RMAN Backup LEVEL 0"

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| PERFORM Database Shutdown. |"

    wl "+-------------------------------------------------------------------------+"

    shutdown

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| PERFORM Database rman backup |"

    wl "+-------------------------------------------------------------------------+"

    RMANBackup

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| PERFORM Database Startup. |"

    wl "+-------------------------------------------------------------------------+"

    startup

    # +----------------------------------------------------------------------------+

    # | |

    # | CUSTOM SCRIPT TASKS ( END ) |

    # | |

    # +----------------------------------------------------------------------------+

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| SCAN LOG FILE FOR EXCEPTIONS. IGNORE KNOWN EXCEPTIONS. |"

    wl "+-------------------------------------------------------------------------+"

    egrep 'ORA-|JMA-|RMAN-' $LOG_FILE_NAME | egrep -v 'RMAN-20242|RMAN-00571|RMAN-00569|RMAN-03002'

    if (( $? == 0 ))

    then

    wl " "

    wl "+----------------------------------------------+"

    wl "| !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!|"

    wl "| |"

    wl "| ---> ERRORS WERE FOUND <--- |"

    wl "| |"

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

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| REMOVING OBSOLETE SCRIPT LOG FILES (greater than 8 days old)"

    wl "| (${CUSTOM_ORACLE_LOG_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}_*.log)"

    wl "+-------------------------------------------------------------------------+"

    find ${CUSTOM_ORACLE_LOG_DIR}/ -name "${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}_*.log" -mtime +8 -exec ${LS_BIN} -l {} ; | tee -a $LOG_FILE_NAME

    find ${CUSTOM_ORACLE_LOG_DIR}/ -name "${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}_*.log" -mtime +8 -exec ${RM_BIN} -rf {} ; | tee -a $LOG_FILE_NAME

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| ABOUT TO EXIT SCRIPT. |"

    wl "+-------------------------------------------------------------------------+"

    wl " "

    if [[ $ERRORS = "YES" ]]; then

    exitFailed "FAILED"

    else

    exitSuccess "SUCCESSFUL"

    fi
