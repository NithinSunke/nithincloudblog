---
title: "Oracle GoldenGate 21c Installation – Step-by-Step Guide"
author: "Sunke Nithin"
date: 2025-04-26
categories: []
tags: ["Goldengate", "Linux", "technology"]
source: https://nithinsunke.wordpress.com/2025/04/26/oracle-goldengate-21c-installation-step-by-step-guide/
---

# Oracle GoldenGate 21c Installation – Step-by-Step Guide

Oracle GoldenGate 21c is a powerful tool for real-time data replication and integration across heterogeneous environments. This guide will walk you through installing Oracle GoldenGate 21c on a Linux environment.

## Prerequisites

### Supported Operating Systems

- Oracle Linux 7 or 8
- Red Hat Enterprise Linux 7 or 8
- CentOS 7 (not officially supported but can work for testing)

### Software Requirements

- Oracle Database 12c or later
- Oracle GoldenGate 21c binaries (download from oracle edelivery)
- Oracle User with privileges
- Required packages: `glibc`, `libaio`, `unzip`, `zip`, `gcc`, `make`,libnsl
- Java version 11

Step 1: Prepare the Environment

Create OS User and Groups

``` wp-block-code
sudo groupadd oinstall
sudo useradd -m -g oinstall oracle
passwd oracle
```

Create Required Directories

``` wp-block-code
mkdir -p /u01/app/oracle
chown -R oracle:oinstall /u01/app
chmod -R 775 /u01/app
cd /u01/app/oracle
mkdir datasore  ogg21c_deploy1  ogg21c_ma1  ogg21c_sm1
```

## Step 2: Download Oracle GoldenGate 21c

👉 Go to the Oracle Software eDelivery Cloud:

- Select the version for your OS: e.g., `Linux x86-64 (Oracle GoldenGate 21c)`
- Download the ZIP file: `213000_fbo_ggs_Linux_x64_shiphome.zip`

## Step 3: Install Oracle GoldenGate 21c

### Switch to Oracle User

``` wp-block-code
su - oracle
```

### Unzip the Software

``` wp-block-code
unzip 213000_fbo_ggs_Linux_x64_shiphome.zip -d /opt/soft
cd /opt/soft/fbo_ggs_Linux_x64_shiphome/Disk1
```

### Run the Installer

``` wp-block-code
./runInstaller
```

This launches the Oracle Universal Installer (OUI). Follow the GUI wizard.

**Welcome Screen** – Click *Next*

![](images/image-522.png)

**Software Location** – Choose: /u01/app/oracle/ogg21c_ma1

![](images/image-547.png)

click on install

![](images/image-549.png)

![](images/image-551.png)

![](images/image-553.png)

## Steps 4: Create Microservices Deployment

Run the Oracle GoldenGate Configuration Assistant:

``` wp-block-code
cd $OGG_HOME
./oggca.sh
```

GUI Wizard Steps:

**select service manager option:**

![](images/image-528.png)

![](images/image-555.png)

![](images/image-557.png)

![](images/image-559.png)

![](images/image-561.png)

![](images/image-563.png)

![](images/image-565.png)

![](images/image-567.png)

![](images/image-569.png)

![](images/image-572.png)

![](images/image-573.png)

![](images/image-575.png)

![](images/image-577.png)

steps to stop,start the goldengate services

``` wp-block-code
systemctl start OracleGoldenGate
systemctl stop OracleGoldenGate
systemctl status OracleGoldenGate
```

## Step 6: Verify Installation

- Access ServiceManager WebUI.
- Login to AdminServer.
- Check that all services (AdminServer, DistributionServer, ReceiverServer, MetricsServer) are running.
- Start building your Extract and Replicat processes!

![](images/image-579.png)

# Conclusion

That’s it! 🎯  
You have now **successfully installed Oracle GoldenGate 21c Microservices Architecture** on your server.

This setup brings a modern web-based interface for your replication needs and sets the foundation for high availability and efficient data movement.

Stay tuned.......

**Tip**:  
Always keep your GoldenGate Home separate from your Deployment Home to simplify patching and upgrades later.
