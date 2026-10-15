---
title: "Oracle Grid Infrastructure 19c Using Out-Of-Place patching SwitchGridHome"
author: "Sunke Nithin"
date: 2024-06-12
categories: ["Uncategorized"]
tags: ["DATABASE", "databases", "ORACLE", "oracle-database"]
source: https://nithinsunke.wordpress.com/2024/06/12/oracle-grid-infrastructure-19c-using-out-of-place-patching-switchgridhome/
---

# Oracle Grid Infrastructure 19c Using Out-Of-Place patching SwitchGridHome

In this blog we will see how to do oracle grid infrastructure 19c out of place patching by Switching the Grid Home

In out of place patching always using free installation of new home instead of cloning from the existing home.

I have a two node rac with freshly installed. Make sure you have sufficient space to accomidate the new home

### Preparation

1.  download the oracle grid infrastructure 19c binaries
2.  download the latest RU patches 36233126
3.  download latest OPATCH utility

### Prepare a New Grid Home

execute the below steps on all nodes

``` wp-block-code
[oracle@rac1 ~]$ su - root
Password:
Last login: Wed Jun 12 00:47:11 UTC 2024 on pts/0
[root@rac1 ~]# mkdir -p /u01/app/19.23.0/grid
[root@rac1 ~]# chown -R oracle:oinstall /u01/app/19.23.0
[root@rac1 ~]# chmod -R 775 /u01/app/19.23.0
[root@rac1 ~]# su - oracle
Last login: Wed Jun 12 00:50:03 UTC 2024
[oracle@rac1 ~]$ ssh rac1
Last login: Wed Jun 12 00:50:07 2024
[oracle@rac1 ~]$ exit
Logout
```

on node1 unzip the 19c binary file

``` wp-block-code
[oracle@rac1 ~]$ export NEWGRIDHOME=/u01/app/19.23.0/grid
[oracle@rac1 ~]$ cd $NEWGRIDHOME
[oracle@rac1 grid]$ unzip -oq /nfsshare/vagrant-scripts/si/ol7_19/software/LINUX.X64_193000_grid_home.zip
[oracle@rac1 grid]$ pwd
/u01/app/19.23.0/grid

[oracle@rac1 grid]$ cd $NEWGRIDHOME
[oracle@rac1 grid]$ rm -rf OPatch
[oracle@rac1 grid]$ unzip -oq /u01/patch/p6880880_122010_Linux-x86-64.zip
[oracle@rac1 grid]$ ls
```

as grid owner run the prereqs

``` wp-block-code
[oracle@rac1 grid]$ export ORACLE_HOME=$NEWGRIDHOME
[oracle@rac1 grid]$ $ORACLE_HOME/gridSetup.sh -executePrereqs -silent
Launching Oracle Grid Infrastructure Setup Wizard...

[WARNING] [INS-13014] Target environment does not meet some optional requirements.
   CAUSE: Some of the optional prerequisites are not met. See logs for details. /u01/app/oraInventory/logs/GridSetupActions2024-06-12_00-56-28AM/gridSetupActions2024-06-12_00-56-28AM.log
   ACTION: Identify the list of failed prerequisite checks from the log: /u01/app/oraInventory/logs/GridSetupActions2024-06-12_00-56-28AM/gridSetupActions2024-06-12_00-56-28AM.log. Then either from the log file or from installation manual find the appropriate configuration to meet the prerequisites and fix it manually.
[oracle@rac1 grid]$ vi /u01/app/oraInventory/logs/GridSetupActions2024-06-12_00-56-28AM/gridSetupActions2024-06-12_00-56-28AM.log
```

in my case the above warning is due to insufficient swap space we will ignore for now.

we will install new grid home by applying the latest RU patches at same time.

``` wp-block-code
[oracle@rac1 grid]$ export ORACLE_BASE=/u01/app/grid
[oracle@rac1 grid]$ export ORA_INVENTORY=/u01/app/oraInventory
[oracle@rac1 grid]$ export CLUSTER_NAME=$(olsnodes -c)
[oracle@rac1 grid]$ export CLUSTER_NODES=$(olsnodes | tr '\n' ','| sed 's/,\s*$//')
[oracle@rac1 grid]$ cd $ORACLE_HOME
[oracle@rac1 grid]$ ./gridSetup.sh -ignorePrereq -waitforcompletion -silent \
>    -applyRU /u01/patch/36233126 \
>    -responseFile $ORACLE_HOME/install/response/gridsetup.rsp \
>    INVENTORY_LOCATION=$ORA_INVENTORY \
>    ORACLE_BASE=$ORACLE_BASE \
>    SELECTED_LANGUAGES=en \
>    oracle.install.option=CRS_SWONLY \
>    oracle.install.asm.OSDBA=asmdba \
>    oracle.install.asm.OSOPER=asmoper \
>    oracle.install.asm.OSASM=asmadmin \
>    oracle.install.crs.config.ClusterConfiguration=STANDALONE \
>    oracle.install.crs.config.configureAsExtendedCluster=false \
>    oracle.install.crs.config.clusterName=$CLUSTER_NAME \
>    oracle.install.crs.config.gpnp.configureGNS=false \
>    oracle.install.crs.config.autoConfigureClusterNodeVIP=false \
>    oracle.install.crs.config.clusterNodes=$CLUSTER_NODES
Preparing the home to patch...
Applying the patch /u01/patch/36233126...
Successfully applied the patch.
The log can be found at: /u01/app/oraInventory/logs/GridSetupActions2024-06-12_01-05-02AM/installerPatchActions_2024-06-12_01-05-02AM.log
Launching Oracle Grid Infrastructure Setup Wizard...

[WARNING] [INS-13014] Target environment does not meet some optional requirements.
   CAUSE: Some of the optional prerequisites are not met. See logs for details. /u01/app/oraInventory/logs/GridSetupActions2024-06-12_01-05-02AM/gridSetupActions2024-06-12_01-05-02AM.log
   ACTION: Identify the list of failed prerequisite checks from the log: /u01/app/oraInventory/logs/GridSetupActions2024-06-12_01-05-02AM/gridSetupActions2024-06-12_01-05-02AM.log. Then either from the log file or from installation manual find the appropriate configuration to meet the prerequisites and fix it manually.
The response file for this session can be found at:
 /u01/app/19.23.0/grid/install/response/grid_2024-06-12_01-05-02AM.rsp

You can find the log of this install session at:
 /u01/app/oraInventory/logs/GridSetupActions2024-06-12_01-05-02AM/gridSetupActions2024-06-12_01-05-02AM.log

As a root user, execute the following script(s):
        1. /u01/app/19.23.0/grid/root.sh

Execute /u01/app/19.23.0/grid/root.sh on the following nodes:
[rac1, rac2]

Successfully Setup Software with warning(s).
```

dont run the root.sh now we will run it once we switch the grid home

### Switch to the new Grid Home

run the switch home script on both the nodes

``` wp-block-code
[oracle@rac1 grid]$ export ORACLE_HOME=/u01/app/19.23.0/grid
[oracle@rac1 grid]$ export CURRENT_NODE=$(hostname)
[oracle@rac1 grid]$ $ORACLE_HOME/gridSetup.sh \
>    -silent -switchGridHome \
>    oracle.install.option=CRS_SWONLY \
>    ORACLE_HOME=$ORACLE_HOME \
>    oracle.install.crs.config.clusterNodes=$CURRENT_NODE \
>    oracle.install.crs.rootconfig.executeRootScript=false
Launching Oracle Grid Infrastructure Setup Wizard...
As a root user, execute the following script(s):
        1. /u01/app/19.23.0/grid/root.sh

Execute /u01/app/19.23.0/grid/root.sh on the following nodes:
[rac1]

You can find the log of this install session at:
 /u01/app/oraInventory/logs/UpdateNodeList2024-06-12_02-01-42AM.log
You can find the log of this install session at:
 /u01/app/oraInventory/logs/UpdateNodeList2024-06-12_02-01-42AM.log
Successfully Setup Software.
[oracle@rac1 grid]$ $ORACLE_HOME/gridSetup.sh    -silent -switchGridHome    oracle.install.option=CRS_SWONLY    ORACLE_HOME=$ORACLE_HOME    oracle.install.crs.config.clusterNodes=$CURRENT_NODE    oracle.install.crs.rootconfig.executeRootScript=false
Launching Oracle Grid Infrastructure Setup Wizard...

As a root user, execute the following script(s):
        1. /u01/app/19.23.0/grid/root.sh

Execute /u01/app/19.23.0/grid/root.sh on the following nodes:
[rac1]

You can find the log of this install session at:
 /u01/app/oraInventory/logs/UpdateNodeList2024-06-12_02-02-16AM.log
You can find the log of this install session at:
 /u01/app/oraInventory/logs/UpdateNodeList2024-06-12_02-02-16AM.log
Successfully Setup Software.
[oracle@rac1 grid]$
```

now run the root .sh

``` wp-block-code
[oracle@rac1 grid]$ su - root
Password:
Last login: Wed Jun 12 00:48:33 UTC 2024 on pts/0
[root@rac1 ~]# /u01/app/19.23.0/grid/root.sh
Check /u01/app/19.23.0/grid/install/root_rac1.scslabs.com_2024-06-12_02-06-44-081548068.log for the output of root script
```

now update the env profiles. to point to new Grid home

Clean the Old Grid Home inorder to freeup the space

we will use deinstall tool to deinstall the old grid home need to run on both nodes one by one

``` wp-block-code
$ export ORACLE_HOME=/u01/app/19.0.0.0/grid
$ $ORACLE_HOME/deinstall/deinstall -local
```

But dont do it on the same day monitor for a week and clean the old grid home whoever it wont require any downtime.
