---
title: "Oracle Dataguard Configuration shell script"
author: "Sunke Nithin"
date: 2019-07-09
categories: ["SHELL SCRIPTS"]
tags: []
source: https://nithinsunke.wordpress.com/2019/07/09/oracle-dataguard-configuration-shell-script/
---

# Oracle Dataguard Configuration shell script

#!/bin/sh
    ORACLE_SID=sunkdev
    ORAENV_ASK=NO
    . oraenv
    ORAENV_ASK=YES

    # +----------------------------------------------------------------------------+

    # | ************************************************************************** |

    # | * DEFINE ALL GLOBAL VARIABLES * |

    # | ************************************************************************** |

    # +----------------------------------------------------------------------------+

    # ----------------------------

    # SCRIPT NAME VARIABLES

    # ----------------------------

    SCRIPT_NAME_FULL=$0

    SCRIPT_NAME=${SCRIPT_NAME_FULL##*/}

    SCRIPT_NAME_NOEXT=${SCRIPT_NAME%%.sh}

    # ----------------------------

    # CUSTOM SCRIPT VARIABLES

    # ----------------------------

    PRIMARY_DB_NAME=sunkdev

    STANDBY_DB_NAME=sunksbdev

    DBA_USERNAME=SYS

    DBA_PASSWORD='<sys_password>'

    PRIMARY_HOST_NAME=oradbdvl01.snv.ind

    STANDBY_HOST_NAME=orclstdy01.snv.ind

    # ----------------------------

    # ORACLE ENVIRONMENT VARIABLES

    # ----------------------------

    ORACLE_BASE=/a01/oracle

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

    # EMAIL PREFERENCES

    # ----------------------------

    MAIL_RECIPIENT_LIST_EXIT_SUCCESS="dba@example.com"

    MAIL_RECIPIENT_LIST_EXIT_FAILED="dba@example.com"

    MAIL_FROM="${ORGANIZATION_NAME} Database Support <dba@example.com>"

    MAIL_REPLYTO="${ORGANIZATION_NAME} Database Support <dba@example.com>"

    MAIL_TO_NAME="${ORGANIZATION_NAME} Database Support"

    MAIL_TEMP_FILE_NAME=${CUSTOM_ORACLE_TEMP_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}.mhr

    # ----------------------------

    # DATE VARIABLES

    # ----------------------------

    START_DATE=`date`

    DATE_LOG=`date +%Y-%m-%d-%H:%M:%S`

    TODAY=${DATE_LOG}

    # ----------------------------

    # HOSTNAME VARIABLES

    # ----------------------------

    HOSTNAME=`hostname`

    HOSTNAME_UPPER=`echo $HOSTNAME | tr '[:lower:]' '[:upper:]'`

    HOSTNAME_SHORT=${HOSTNAME%%.*}

    HOSTNAME_SHORT_UPPER=`echo $HOSTNAME_SHORT | tr '[:lower:]' '[:upper:]'`

    # ----------------------------

    # LOG FILE VARIABLES

    # ----------------------------

    LOG_FILE_NAME=${CUSTOM_ORACLE_LOG_DIR}/${SCRIPT_NAME_NOEXT}_${2}_${DATE_LOG}.log

    LOG_FILE_NAME_NODATE=${CUSTOM_ORACLE_LOG_DIR}/${SCRIPT_NAME_NOEXT}_${2}.log

    LOG_FILE_ARCHIVE_OBSOLETE_DAYS=7

    CHECK_SCRIPT_RUNNING_FLAG_FILE=${CUSTOM_ORACLE_TEMP_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}.running

    SQL_OUTPUT_TEMP_FILE_NAME=${CUSTOM_ORACLE_TEMP_DIR}/${SCRIPT_NAME_NOEXT}_${HOSTNAME_SHORT_UPPER}.lst

    # ----------------------------

    # MISCELLANEOUS VARIABLES

    # ----------------------------

    HOST_RVAL_SUCCESS=0

    HOST_RVAL_FAILED=1

    HIDE_PASSWORD_STRING="xxxxxxxxxxxxx"

    # +----------------------------------------------------------------------------+

    # | ************************************************************************** |

    # | * DEFINE ALL GLOBAL FUNCTIONS * |

    # | ************************************************************************** |

    # +----------------------------------------------------------------------------+

    function wl {

    echo "${1}" >> ${LOG_FILE_NAME}

    echo "${1}"

    }

    function showUsage {

    wl "Usage: ${SCRIPT_NAME}"

    wl " "

    }

    function startLogging {

    wl "+=========================================================================+"

    wl "| |"

    wl "| START TIME |"

    wl "| |"

    wl "| $START_DATE |"

    wl "| |"

    wl "+=========================================================================+"

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

    function ARCHIVE_Logging {

    sqlplus /nolog <<eof

    conn / as sysdba

    set head off

    spool $SQL_OUTPUT_TEMP_FILE_NAME

    select log_mode FROM v$database;

    spool off

    exit

    eof

    ARCHIVE_STATUS=`cat $SQL_OUTPUT_TEMP_FILE_NAME |grep -i "ARCHIVELOG" |tr -d "/t/n/r"|tr -d "[:space:]"`

    if [ "$ARCHIVE_STATUS" == "ARCHIVELOG" ]

    then

    wl " "

    wl " "

    wl "----------------------------"

    wl " Database in Archivelog Mode"

    wl "----------------------------"

    else

    wl " "

    wl " "

    wl "------------------------------"

    wl " Database in NOarchivelog Mode"

    wl "------------------------------"

    wl " "

    wl " Do you want to enable archivelog mode (y/n):"

    read userResponse

    if [[ ${userResponse} == "" || ${userResponse} == "N" || ${userResponse} == "n" ]]; then

    wl "Exiting script as per user request."

    wl "Good-bye!"

    wl " "

    exit

    else

    sqlplus /nolog <<eof

    conn / as sysdba

    shutdown immediate

    exit

    eof

    wl "Database is Shutdown"

    sqlplus /nolog <<eof

    conn / as sysdba

    startup mount

    exit

    eof

    wl "Database is mounted"

    sqlplus /nolog <<eof

    conn / as sysdba

    alter database archivelog;

    alter database open;

    exit

    eof

    sqlplus /nolog <<eof

    conn / as sysdba

    set head off

    spool $SQL_OUTPUT_TEMP_FILE_NAME

    select log_mode FROM v$database;

    spool off

    exit

    eof

    ARCHIVE_STATUS=`cat $SQL_OUTPUT_TEMP_FILE_NAME |grep -i "ARCHIVELOG" |tr -d "/t/n/r"|tr -d "[:space:]"`

    wl "====================================="

    wl "Database in $ARCHIVE_STATUS Mode "

    wl "====================================="

    fi

    fi

    }

    function force_logging {

    sqlplus /nolog <<eof

    conn / as sysdba

    spool $SQL_OUTPUT_TEMP_FILE_NAME

    select FORCE_LOGGING from v$database;

    spool off

    exit

    eof

    FL_STATUS=`cat $SQL_OUTPUT_TEMP_FILE_NAME |grep -i "YES"|wc -l`

    echo $FL_STATUS

    if [ $FL_STATUS -eq 0 ]

    then

    wl "-----------------------------"

    wl "FORCE LOGGING IS NOT ENABLED "

    wl "-----------------------------"

    wl " Do you want to enable force logging in primary database (y/n):"

    read userResponse

    if [[ ${userResponse} == "" || ${userResponse} == "N" || ${userResponse} == "n" ]]; then

    wl "Exiting script as per user request."

    wl "Good-bye!"

    wl " "

    exit

    else

    sqlplus /nolog <<eof

    conn / as sysdba

    ALTER DATABASE FORCE LOGGING;

    exit

    eof

    fi

    else

    wl "-------------------------"

    wl "FORCE LOGGING IS ENABLED "

    wl "-------------------------"

    fi

    }

    function initialization_params {

    wl "Configure initialization parameters to build standby database in primary database"

    wl "ALTER SYSTEM SET LOG_ARCHIVE_CONFIG='DG_CONFIG=($PRIMARY_DB_NAME",$STANDBY_DB_NAME")' scope=spfile;"

    wl "ALTER SYSTEM SET LOG_ARCHIVE_DEST_2='SERVICE=$STANDBY_DB_NAME NOAFFIRM ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=$STANDBY_DB_NAME' scope=spfile;"

    wl "ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=ENABLE scope=spfile;"

    wl "ALTER SYSTEM SET FAL_SERVER=$STANDBY_DB_NAME scope=spfile;"

    wl "ALTER SYSTEM SET FAL_CLIENT=$PRIMARY_DB_NAME scope=spfile;"

    wl "ALTER SYSTEM SET STANDBY_FILE_MANAGEMENT=AUTO scope=spfile;"

    wl " "

    wl " "

    wl " Do you want to modify above parameter at primary database(y/n):"

    echo "ALTER SYSTEM SET LOG_ARCHIVE_CONFIG='DG_CONFIG=($PRIMARY_DB_NAME",$STANDBY_DB_NAME")' scope=spfile;

    ALTER SYSTEM SET LOG_ARCHIVE_DEST_2='SERVICE=$STANDBY_DB_NAME NOAFFIRM ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=$STANDBY_DB_NAME' scope=spfile;

    ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=ENABLE scope=spfile;

    ALTER SYSTEM SET FAL_SERVER=$STANDBY_DB_NAME scope=spfile;

    ALTER SYSTEM SET FAL_CLIENT=$PRIMARY_DB_NAME scope=spfile;

    ALTER SYSTEM SET STANDBY_FILE_MANAGEMENT=AUTO scope=spfile;

    " > ${CUSTOM_ORACLE_TEMP_DIR}/primary_confg.sql

    read userResponse

    if [[ ${userResponse} == "" || ${userResponse} == "N" || ${userResponse} == "n" ]]; then

    wl "Exiting script as per user request."

    wl "Good-bye!"

    wl " "

    exit

    else

    sqlplus /nolog <<eof

    conn / as sysdba

    @${CUSTOM_ORACLE_TEMP_DIR}/primary_confg.sql

    exit

    eof

    fi

    }

    function standby_listener {

    wl "

    SID_LIST_LISTENER =

    (SID_LIST =

    (SID_DESC =

    (GLOBAL_DBNAME = $STANDBY_DB_NAME)

    (ORACLE_HOME = /a01/oracle/11.2.0.4/db)

    (SID_NAME = $ORACLE_SID)

    )

    )

    LISTENER =

    (DESCRIPTION_LIST =

    (DESCRIPTION =

    (ADDRESS = (PROTOCOL = TCP)(HOST = $STANDBY_HOST_NAME)(PORT = 1521))

    )

    (DESCRIPTION =

    (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))

    )

    )

    ADR_BASE_LISTENER = /a01/oracle" |tee ${CUSTOM_ORACLE_TEMP_DIR}/listener.ora

    wl "copying listener.ora to standby database server............"

    wl "scp -pr ${CUSTOM_ORACLE_TEMP_DIR}/listener.ora oracle@orclstdy01:/c01/standby"

    scp -pr ${CUSTOM_ORACLE_TEMP_DIR}/listener.ora oracle@orclstdy01:/c01/standby

    ssh oracle@orclstdy01 <<eof | tee -a $LOG_FILE_NAME

    echo "cp /c01/standby/listener.ora /a01/oracle/11.2.0.4/db/network/admin/listener.ora"

    cp /c01/standby/listener.ora /a01/oracle/11.2.0.4/db/network/admin/listener.ora

    ORACLE_SID=sunkdev

    ORAENV_ASK=NO

    . oraenv

    ORAENV_ASK=YES

    lsnrctl start

    eof

    }

    function tnsnames {

    wl "$PRIMARY_DB_NAME =

    (DESCRIPTION =

    (ADDRESS_LIST =

    (ADDRESS = (PROTOCOL = TCP)(HOST = $PRIMARY_HOST_NAME)(PORT = 1521))

    )

    (CONNECT_DATA =

    (SERVICE_NAME = $PRIMARY_DB_NAME)

    )

    )

    $STANDBY_DB_NAME =

    (DESCRIPTION =

    (ADDRESS_LIST =

    (ADDRESS = (PROTOCOL = TCP)(HOST = $STANDBY_HOST_NAME)(PORT = 1521))

    )

    (CONNECT_DATA =

    (SERVICE_NAME = $STANDBY_DB_NAME)

    )

    )

    " |tee ${CUSTOM_ORACLE_TEMP_DIR}/tnsnames.ora

    wl "copying script generated tnsentries to tnsnames.ora"

    wl "cat ${CUSTOM_ORACLE_TEMP_DIR}/tnsnames.ora >> $ORACLE_HOME/network/admin/tnsnames.ora"

    cat ${CUSTOM_ORACLE_TEMP_DIR}/tnsnames.ora >> $ORACLE_HOME/network/admin/tnsnames.ora

    wl " "

    #setup password less ssh between two hosts

    #use link https://oracle-base.com/articles/linux/user-equivalence-configuration-on-linux

    wl "copying tnsnames.ora to standby database server............"

    wl "scp -pr ${CUSTOM_ORACLE_TEMP_DIR}/tnsnames.ora oracle@orclstdy01:/c01/standby"

    scp -pr ${CUSTOM_ORACLE_TEMP_DIR}/tnsnames.ora oracle@orclstdy01:/c01/standby

    ssh oracle@orclstdy01 <<eof | tee -a $LOG_FILE_NAME

    echo "cat /c01/standby/tnsnames.ora >> /a01/oracle/11.2.0.4/db/network/admin/tnsnames.ora"

    cat /c01/standby/tnsnames.ora >> /a01/oracle/11.2.0.4/db/network/admin/tnsnames.ora

    eof

    }

    function standby_pfile {

    echo "db_name=sunkdev

    db_unique_name=sunksbdev" > ${CUSTOM_ORACLE_TEMP_DIR}/initsunkdev.ora

    scp -pr ${CUSTOM_ORACLE_TEMP_DIR}/initsunkdev.ora oracle@orclstdy01:/a01/oracle/11.2.0.4/db/dbs

    }

    function standby_redo {

    wl " do you want to create standby redologs (y/n):"

    read userResponse

    if [[ ${userResponse} == "" || ${userResponse} == "N" || ${userResponse} == "n" ]]; then

    wl "Exiting script as per user request."

    wl "Good-bye!"

    wl " "

    exit

    else

    sqlplus /nolog <<eof |tee -a $LOG_FILE_NAME

    conn / as sysdba

    ALTER DATABASE ADD LOGFILE ('/a01/oracle/oradata/sunkdev/stdby_redo01.log') SIZE 50M;

    ALTER DATABASE ADD LOGFILE ('/a01/oracle/oradata/sunkdev/stdby_redo02.log') SIZE 50M;

    ALTER DATABASE ADD LOGFILE ('/a01/oracle/oradata/sunkdev/stdby_redo03.log') SIZE 50M;

    ALTER DATABASE ADD LOGFILE ('/a01/oracle/oradata/sunkdev/stdby_redo04.log') SIZE 50M;

    ALTER SYSTEM SET STANDBY_FILE_MANAGEMENT=AUTO;

    exit

    eof

    fi

    }

    function rman_duplicate {

    echo "DUPLICATE TARGET DATABASE

    FOR STANDBY

    FROM ACTIVE DATABASE

    DORECOVER

    SPFILE

    SET db_unique_name='${STANDBY_DB_NAME}' COMMENT 'Is standby'

    SET LOG_ARCHIVE_DEST_2='SERVICE=${PRIMARY_DB_NAME} ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=${PRIMARY_DB_NAME}'

    SET FAL_SERVER='${PRIMARY_DB_NAME}' COMMENT 'Is primary'

    NOFILENAMECHECK; " >> ${CUSTOM_ORACLE_TEMP_DIR}/active_duplicate.cmd

    scp -pr ${CUSTOM_ORACLE_TEMP_DIR}/active_duplicate.cmd oracle@orclstdy01:/c01/standby

    ssh oracle@orclstdy01 <<eof | tee -a $LOG_FILE_NAME

    ORACLE_SID=sunkdev

    ORAENV_ASK=NO

    . oraenv

    ORAENV_ASK=YES

    export PRIMARY_DB_NAME=sunkdev

    export STANDBY_DB_NAME=sunksbdev

    rman target $DBA_USERNAME/$DBA_PASSWORD@$PRIMARY_DB_NAME auxiliary $DBA_USERNAME/$DBA_PASSWORD@$STANDBY_DB_NAME <<abc

    @/c01/standby/active_duplicate.cmd

    exit

    abc

    eof

    }

    function mrp_process {

    ssh oracle@orclstdy01 <<abc| tee -a $LOG_FILE_NAME

    ORACLE_SID=sunkdev

    ORAENV_ASK=NO

    . oraenv

    ORAENV_ASK=YES

    export PRIMARY_DB_NAME=sunkdev

    export STANDBY_DB_NAME=sunksbdev

    sqlplus /nolog <<eof

    conn / as sysdba

    ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT FROM SESSION;

    exit

    eof

    abc

    }

    function start_standby_db {

    ssh oracle@orclstdy01 <<abc | tee -a $LOG_FILE_NAME

    ORACLE_SID=sunkdev

    ORAENV_ASK=NO

    . oraenv

    ORAENV_ASK=YES

    sqlplus /nolog <<eof

    conn / as sysdba

    startup nomount pfile='/a01/oracle/11.2.0.4/db/dbs/initsunkdev.ora'

    exit

    eof

    abc

    }

    # +----------------------------------------------------------------------------+

    # | |

    # | SCRIPT STARTS HERE |

    # | |

    # +----------------------------------------------------------------------------+

    initializeScript

    # +----------------------------------------------------------------------------+

    # | VERIFY AN INSTANCE OF THIS SCRIPT IS NOT ALREADY RUNNING |

    # +----------------------------------------------------------------------------+

    checkScriptAlreadyRunning

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| DISPLAY ALL SCRIPT ENVIRONMENT VARIABLES. |"

    wl "+-------------------------------------------------------------------------+"

    wl "==================================================================================="

    wl " GLOBAL SCRIPT VARIABLES"

    wl "==================================================================================="

    wl "SCRIPT : $SCRIPT_NAME"

    wl "START TIME : $START_DATE"

    wl "HOST NAME : $HOSTNAME"

    wl "LOG_FILE_NAME : $LOG_FILE_NAME"

    wl "LOG_FILE_NAME_NODATE : $LOG_FILE_NAME_NODATE"

    wl "LOG_FILE_ARCHIVE_OBSOLETE_DAYS : $LOG_FILE_ARCHIVE_OBSOLETE_DAYS"

    wl "==================================================================================="

    wl " CUSTOM SCRIPT VARIABLES"

    wl "==================================================================================="

    wl "STANDBY_SID : $ORACLE_SID"

    wl "STANDBY_DB_NAME : $STANDBY_DB_NAME"

    wl "STANDBY_SYS_PASSWORD : xxxxxxxxxxxx"

    wl "PRIMARY_HOST_NAME : $PRIMARY_HOST_NAME"

    wl "STANDBY_HOST_NAME : $STANDBY_HOST_NAME"

    wl "DBA_USERNAME : $DBA_USERNAME"

    wl "==================================================================================="

    wl " "

    wl " "

    wl " "

    wl "+-----------------------+"

    wl "| !!! WARNING !!! |"

    wl "|------------------------------------------------------------------------+"

    wl "| This script is responsible for creating a physical standby database |"

    wl "| for the primary production database running on this node. The standby |"

    wl "| will be created using an RMAN backup of the primary database that will |"

    wl "| be created and transferred to the standby site. |"

    wl "| |"

    wl "| Please note that the following prerequisites MUST be met before |"

    wl "| continuing the running of this script: |"

    wl "| |"

    wl "| 1.) RMAN must be configured with persistant configuration "

    wl "| parameters for the primary database (${TARGET_DB}) "

    wl "| --> PASSED "

    wl "| "

    wl "| 2.) The auxiliary instance (${TARGET_DB}) should be running "

    wl "| in NOMOUNT mode on host ${REMOTE_COPY_SERVER}. "

    wl "| --> PASSED "

    wl "| "

    wl "| 3.) The auxiliary instance (${TARGET_DB}) should be accessible "

    wl "| through the TNS service name (${AUXILIARY_DB}). "

    wl "| --> PASSED "

    wl "| "

    wl "| 4.) The auxiliary instance (${TARGET_DB}) has a valid password "

    wl "| file on host ${REMOTE_COPY_SERVER}. "

    wl "| --> PASSED "

    wl "| "

    wl "| 5.) The auxiliary instance (${TARGET_DB}) has a valid SPFILE "

    wl "| on host ${REMOTE_COPY_SERVER}. "

    wl "| --> PASSED "

    wl "| "

    wl "+------------------------------------------------------------------------+"

    wl " "

    wl "Would you like to continue to create the standby database (y/[n])?"

    wl " "

    read userResponse

    if [[ ${userResponse} == "" || ${userResponse} == "N" || ${userResponse} == "n" ]]; then

    wl "Exiting script as per user request."

    wl "Good-bye!"

    wl " "

    exit

    fi

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "| Performing configuration steps on standby.............. |"

    wl "+-------------------------------------------------------------------------+"

    wl " "

    wl "+----------------------------------------------------------------------------+"

    wl "| TEST LOGIN TO PRIMARY DATABASE |"

    wl "+----------------------------------------------------------------------------+"

    verifyDatabaseLoginCredentials $PRIMARY_DB_NAME $DBA_USERNAME $DBA_PASSWORD SYSDBA

    wl "Check that the primary database is in archivelog mode. "

    ARCHIVE_Logging

    wl " "

    wl " "

    wl "Check that the primary database is in force logging mode"

    force_logging

    wl " "

    wl " "

    wl "configuring initialization parameters"

    initialization_params

    wl " "

    DATE_PRINT_LOG=`date "+%m/%d/%Y %H:%M:%S"`

    wl " "

    wl "+-------------------------------------------------------------------------+"

    wl "| ${DATE_PRINT_LOG} |"

    wl "|-------------------------------------------------------------------------|"

    wl "Creating Directory Structure at standby"

    ssh $STANDBY_HOST_NAME <<eof | tee -a $LOG_FILE_NAME

    mkdir -p /a01/oracle/oradata/sunkdev

    mkdir -p /a01/oracle/fast_recovery_area/SUNKSBDEV

    eof

    wl "Configure standby listener and start the listener at standby"

    standby_listener

    wl " verify tns connectivity between primary and standby "

    tnsnames

    wl " "

    wl "==========================="

    wl "on primary database server "

    wl "==========================="

    wl " "

    wl " "

    verifyTNSConnectString $PRIMARY_DB_NAME

    verifyTNSConnectString $STANDBY_DB_NAME

    wl " "

    wl "============================"

    wl " on standby database server "

    wl "============================"

    ssh oracle@orclstdy01 <<eof| tee -a $LOG_FILE_NAME

    ORACLE_SID=sunkdev

    ORAENV_ASK=NO

    . oraenv

    ORAENV_ASK=YES

    export PRIMARY_DB_NAME=sunkdev

    export STANDBY_DB_NAME=sunksbdev

    tnsping $PRIMARY_DB_NAME

    if (( $? == 0 )); then

    echo " "

    echo "TRACE> The TNS service name for the auxiliary instance (${PRIMARY_DB_NAME}) IS valid!"

    else

    echo " "

    echo "TRACE> The TNS service name for the auxiliary instance (${PRIMARY_DB_NAME}) IS NOT valid."

    echo " "

    errorExit

    fi

    tnsping $STANDBY_DB_NAME

    if (( $? == 0 )); then

    echo " "

    echo "TRACE> The TNS service name for the auxiliary instance (${STANDBY_DB_NAME}) IS valid!"

    else

    echo " "

    echo "TRACE> The TNS service name for the auxiliary instance (${STANDBY_DB_NAME}) IS NOT valid."

    echo " "

    errorExit

    fi

    eof

    wl "Configure Standby redologs at primary database"

    standby_redo

    wl " Configure standby database pfile "

    standby_pfile

    wl " "

    wl "+----------------------------------------------------------------------------+"

    wl "| PUT STANDBY DATABASE IN NOMOUNT MODE |"

    wl "+----------------------------------------------------------------------------+"

    start_standby_db

    wl " "

    wl "+----------------------------------------------------------------------------+"

    wl "| TEST LOGIN TO STANDBY DATABASE |"

    wl "+----------------------------------------------------------------------------+"

    verifyDatabaseLoginCredentials $STANDBY_DB_NAME $DBA_USERNAME $DBA_PASSWORD SYSDBA

    wl " "

    wl "+----------------------------------------------------------------------------+"

    wl "| RECOVER / CREATE STANDBY DATABASE |"

    wl "| -------------------------------------------------------------------------- |"

    wl "| Login to target (primary) and (standby) auxiliary database using RMAN. All |"

    wl "| of this should be performed from the target database server. In order to |"

    wl "| perform this section, you will need the last log sequence number, which |"

    wl "| we recorded earlier in this script. |"

    wl "| |"

    wl "| Notice that the parameter NOFILENAMECHECK must be used when you are |"

    wl "| duplicating a database to a different host with the same file system |"

    wl "| (directory structure). |"

    wl "+----------------------------------------------------------------------------+"

    rman_duplicate

    wl " "

    wl "+----------------------------------------------------------------------------+"

    wl "| PUT STANDBY DATABASE IN MANAGED STANDBY MODE |"

    wl "+----------------------------------------------------------------------------+"

    mrp_process

    wl " "

    wl "+----------------------------------------------------------------------------+"

    wl "| DO A FINAL LOG SWITCH ON PRIMARY |"

    wl "| -------------------------------------------------------------------------- |"

    wl "| This is needed to re-establish connection to standby database in max. |"

    wl "| avail. mode. |"

    wl "+----------------------------------------------------------------------------+"

    $ORACLE_HOME/bin/sqlplus /nolog <<EOF | tee -a $LOG_FILE_NAME

    connect ${DBA_USERNAME}/${DBA_PASSWORD}@${PRIMARY_DB_NAME} as sysdba

    set heading off

    alter system set log_archive_dest_state_2 = enable scope=both;

    alter system switch logfile;

    EOF

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

    egrep 'ORA-|JMA-|RMAN-' $LOG_FILE_NAME | egrep -v 'RMAN-20242|RMAN-00571|RMAN-00569|RMAN-03002|RMAN-07529|RMAN-07517|RMAN-05538'

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
