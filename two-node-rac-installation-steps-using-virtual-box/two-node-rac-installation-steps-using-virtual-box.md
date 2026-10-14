---
title: "TWO NODE RAC INSTALLATION STEPS USING VIRTUAL BOX"
author: "Sunke Nithin"
date: 2019-06-26
categories: ["RAC"]
tags: []
source: https://nithinsunke.wordpress.com/2019/06/26/two-node-rac-installation-steps-using-virtual-box/
---

# TWO NODE RAC INSTALLATION STEPS USING VIRTUAL BOX

**ORACLE LINUX 5.6 INSTALLATION SETUP:**

Insert Installation Media \#1:

Testing the CD Media: Skip

Language: English

Key Board: U.S. English

Partition Option: “Remove all Partitions on selected drives and create default layout”

Boot Loader: “ The GRUB boot loader will be installed on /dev/sda”

**Network Devices:**  

  

  

  

Active on Boot

  

Devices

  

IPV4.Netmask

  

IPV6/Prefix

  

  

  

Yes

  

eth0

  

192.168.1.11/255.255.255.0

  

Disable

  

  

  

Yes

  

eth1

  

192.168.2.11/255.255.255.0

  

Disable

  

  

  

  

    Hostname : Manually : rac1.oracle.com

  
Ignore both the Warning Messages at this point

Region: America/New York

System Clock Uses UTC (checked)

Root Password  Enter the root password

Additional Tasks On top of Default Installation:  “Checked all Software Development” and “Web Server”

Customize Now (Selected)

(Below is the extra selection on top of the default selected packages)  

    Applications -> Authoring and Publishing (checked)

    Development -> Development Libraries ->libstdc++44-devel

    Development -> Java Development

    Development -> Legacy Software Development

    Servers -> Checked All the servers

    Servers -> Legacy Network Server ->bootparamd, rsh-server, rusers, rusers-server, telnet-server

    Servers -> Network Servers ->dhcp, dhcpv6, dnsmasq, ypserv

    Servers -> Servers Configuration Tools ->Checked All

    Base System ->Administration Tools ->Checked All

    Base System ->Base ->device-mapper-multipath, iscsi-initiator-utils,

    Base System ->Legacy Software Support ->openmotif22

    Base System -> System Tools ->OpenIPMI-gui, lsscsi, oracle*, sysstat, tsclient

  
 

**Post Installation Steps:**  

    (1)    Yes to License Agreement.

    (2)    Disable the firewall

    (3)    Disable SELinux

    (4)    Disable kdump

    (5)    Set the clock

    (6)    Finish

  
Asks for reboot.

 

After restart login as root user and perform below steps.  

    Service iptables stop

    Chkconfig iptables off

    Chkconfig –list iptables

    Vim /etc/selinux/config   à should be disabled

  
 

**Steps to configure yum:**

Go to  /etc/yum.repos.d  

    cd /etc/yum.repos.d à type below entries and hit enter a file get created in the directory.

    wget http://public-yum.oracle.com/public-yum-ol6.repo

  
 

check the prerequisites for installation of oracle rdbms.  

    Yum install –y oracle-rdbms-server-11gR2-Preinstall

    Yum install –y man partcd.x86_64 unzip.x86_64 xterm lsof bind xorg-x11-twm

    Yum remove –y ntp

  
 

**Step by step DNS Configuration on Linux RHEL5/OEL5**

**Step 1 : Login as root and install below listed Bind RPM's.**

Check the following rpm are installed in os or not using command

Rpm –qa \<rpm name\>  

    rpm -Uvh bind-9*

    rpm -Uvh bind-chroot*

    rpm -Uvh bind-devel-9.*

    rpm -Uvh bind-libbind-*

    rpm -Uvh bind-libs-*

    rpm -Uvh bind-sdb-9.*

    rpm -Uvh system-config-bind-*

    rpm -Uvh caching-nameserver-*

    rpm -Uvh postgresql-libs-*

  
**Step 2: Verify the IP address for the DNS server. Use following command to check the IP address.**  

    [root@dnc ~]# ifconfig

    eth0 Link encap:Ethernet HWaddr 00:0C:29:7D:60:F3

    inet addr:192.168.1.11 Bcast:192.168.1.255 Mask:255.255.255.0

    inet6 addr: fe80::20c:29ff:fe7d:60f3/64 Scope:Link

    UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1

    RX packets:28 errors:0 dropped:0 overruns:0 frame:0

    TX packets:39 errors:0 dropped:0 overruns:0 carrier:0

    collisions:0 txqueuelen:1000

    RX bytes:4919 (4.8 KiB) TX bytes:5628 (5.4 KiB)

    Interrupt:67 Base address:0x2024

  
**Step 3: There are four files we have to edit. Find the below easy steps to configure the DNS server.**  

1.  **Create a named.conf file using sample named.caching-nameserver.conf file.**

  

    [root@dnc ~]# cd /var/named/chroot/etc/

    [root@dnc etc]# ls

    localtime named.caching-nameserver.conf named.rfc1912.zones rndc.key

    [root@dnc etc]#cp named.caching-nameserver.conf named.conf

  
   

2.  **Edit the named.conf file based on your configuration. For example.**

  
   

    options {

    listen-on port 53 { 192.168.1.11; };

    # listen-on-v6 port 53 { ::1; };

    directory "/var/named";

    dump-file "/var/named/data/cache_dump.db";

    statistics-file "/var/named/data/named_stats.txt";

    memstatistics-file "/var/named/data/named_mem_stats.txt";

    // Those options should be used carefully because they disable port

    // randomization

    // query-source port 53;

    // query-source-v6 port 53;

    allow-query { any; };

    allow-query-cache { localhost; };

    };

    logging {

    channel default_debug {

    file "data/named.run";

    severity dynamic;

    };

    };

    view localhost_resolver {

    match-clients { any; };

    match-destinations { 192.168.1.11; };

    recursion yes;

    include "/etc/named.rfc1912.zones";

    };

  

3.  **Edit named.rfc1912.zones file. Sample file given below.**

  

    zone "." IN {

    type hint;

    file "named.ca";

    };

    zone "oracle.com" IN {

    type master;

    file "forward.zone";

    allow-update { none; };

    };

    zone "localhost" IN {

    type master;

    file "localhost.zone";

    allow-update { none; };

    };

    zone "1.168.192.in-addr.arpa" IN {

    type master;

    file "reverse.zone";

    allow-update { none; };

    };

  
   

4.  **Change the permission to the above two files.**

  

    chgrp named named.conf

  

5.  **Change the directory to below location**

  

    cd /var/named/chroot/var/named

    cp localdomain.zone forward.zone

    cp named.local reverse.zone

  

6.  **Modify the forward.zone file. example**

  

    $TTL 86400

    @          IN     SOA    rac1.oracle.com.  root.rac1.oracle.com. (

    42 ; serial (d. adams)

    3H ; refresh

    15M ; retry

    1W ; expiry

    1D ) ; minimum

    IN   NS     rac1.oracle.com.

    rac1                       IN   A     192.168.1.11

    rac2            IN   A 192.168.1.12

    rac1-vip     IN   A 192.168.1.41

    rac2-vip                IN   A 192.168.1.42

    rac-scan      IN   A 192.168.1.31

    rac-scan      IN   A 192.168.1.32

    rac-scan      IN   A 192.168.1.33

  
   

7.  **Modify the reverse.zone file**

  

    $TTL 86400

    @        IN     SOA    rac1.oracle.com.    root.rac1.oracle.com. (

    1997022700 ; Serial

    28800 ; Refresh

    14400 ; Retry

    3600000 ; Expire

    86400 ) ; Minimum

    IN   NS   rac1.oracle.com.

    11      IN   PTR               rac1.oracle.com.

    12      IN   PTR               rac2.oracle.com.

    41      IN   PTR               rac1-vip.oracle.com.

    42      IN   PTR               rac2-vip.oracle.com.

    31      IN   PTR               rac-scan.oracle.com.

    32      IN   PTR               rac-scan.oracle.com.

    32      IN   PTR               rac-scan.oracle.com.

  
   

8.  **Change the ownership for both the files.**

  

    chgrp named reverse.zone

    chgrp named forward.zone

  

9.  **make an entry to both /etc/hosts and /etc/resolv.conf files. The samples are below.**

  

    cat /etc/hosts

    192.168.1.11         rac1.oracle.com                        rac1

    192.168.1.12         rac2.oracle.com                        rac2

    192.168.2.11         rac1-priv.oracle.com                rac1-priv

    192.168.2.12         rac2-priv.oracle.com                rac2-priv

    192.168.1.41         rac1-vip.oracle.com                  rac1-vip

    192.168.1.42         rac2-vip.oracle.com                  rac2-vip

    #192.168.1.31         rac-scan.oracle.com                  rac-scan

    #192.168.1.32         rac-scan.oracle.com                  rac-scan

    #192.168.1.33         rac-scan.oracle.com                  rac-scan

  
 

cat /etc/resolve.conf  

    search oracle.com

    namedserver 192.168.1.11

    namedserver 192.168.1.12

  

10. **Restart the named services using below statement.**

  

    service named restart

  
   

11. **Verify the DNS server using below statements**

  

    [root@dnc named]# dig rac1.oracle.com

    ; «» DiG 9.3.6-P1-RedHat-9.3.6-4.P1.el5_4.2 «» rac1.oracle.com

    ;; global options: printcmd

    ;; Got answer:

    ;; -»HEADER«- opcode: QUERY, status: NOERROR, id: 1483

    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 0

    ;; QUESTION SECTION:

    ;rac1.oracle.com. IN A

    ;; ANSWER SECTION:

    rac1.oracle.com. 86400 IN A 192.168.1.11

    ;; AUTHORITY SECTION:

    india.com. 86400 IN NS dnc.india.com.

    ;; Query time: 1 msec

    ;; SERVER: 192.168.1.11#53(192.168.1.11)

    ;; WHEN: Mon Aug 27 23:54:49 2012

    ;; MSG SIZE rcvd: 61

  
 

**Perform the same for the other**  

    dig rac2.oracle.com

    dig rac1-vip.oracle.com

    dig rac2-vip.oracle.com

    dig rac-scan.oracle.com

  
   

12. **To check reverse the zone**

  

    [root@dnc named]# dig -x 192.168.1.11

    ; «» DiG 9.3.6-P1-RedHat-9.3.6-4.P1.el5_4.2 «» -x 192.168.1.11

    ;; global options: printcmd

    ;; Got answer:

    ;; -»HEADER«- opcode: QUERY, status: NOERROR, id: 55949

    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

    ;; QUESTION SECTION:

    ;11.1.168.192.in-addr.arpa. IN PTR

    ;; ANSWER SECTION:

    11.1.168.192.in-addr.arpa. 86400 IN PTR rac1.oracle.com.

    11.1.168.192.in-addr.arpa. 86400 IN PTR rac1.oracle.com.

    ;; AUTHORITY SECTION:

    1.168.192.in-addr.arpa. 86400 IN NS rac1.oracle.com.

    ;; ADDITIONAL SECTION:

    rac1.oracle.com. 86400 IN A 192.168.1.11

    ;; Query time: 1 msec

    ;; SERVER: 192.168.1.11#53(192.168.1.11)

    ;; WHEN: Mon Aug 27 23:57:27 2012

    ;; MSG SIZE rcvd: 124

  
**Perform the same with others**  

13. **To verify DNS Server using nslookup**

  

    [root@dnc named]# nslookup rac1.oracle.com

    Server: 192.168.1.11

    Address: 192.168.1.11#53

    Name: rac1.oracle.com

    Address: 192.168.1.11

  
DNS server working fine now. Finally we got success.  

14. **Restart named service as a root user.**

  

    [root@dnc named]# service named restart

    Stopping named:                                            [  OK  ]

    Starting named:                                            [  OK  ]

  

15. **Verify the SCAN names using nslookup.**

  

    [root@dnc named]# nslookup rac-scan

    Server:         192.168.1.11

    Address:        192.168.1.11#53

    Name:   rac-scan.oracle.com

    Address: 192.168.1.31

    Name:   rac-scan.oracle.com

    Address: 192.168.1.32

    Name:   rac-scan.oracle.com

    Address: 192.168.1.33

  
 

16\. Add below files in /etc/resolv.conf file on all the nodes.  

    search oracle.com

    nameserver 192.168.1.11

    nameserver 192.168.1.12

  
17. **DNS server ready for Oracle 11gR2 RAC setup go ahead and start RAC installation.**  

    Vi /etc/sysconfig/network-scripts/ifcfg-eth1

    Replace peerdns=yes by no

    Service network restart

    Cat /etc/reslov.conf

  
Check weather the below entries are in /etc/pam.d/login  

    Session                 required      pam-selinux.so               open

    Session                 required       pam-limits.so

  
 

Create group and user for oracle  

    groupadd oinstall

    groupadd dba

    groupadd oper

    usradd –g oinstall –G dba,oper oracle

    mkdir –p /u01/app

    chown        -R  oracle:oinsall   /u01/app

    chmod –R  777     /u01/app

  
**Creating cluster:**

Showtdown the server  

    Shutdown –h now

  
 

**Shared Storage:**

For this example installation we will be using ASM for Clusterware and Database storage on top of SAN technology. The following Table shows the storage layout for this implementation:  

  

  

  

Block Device

  

ASMlib Name

  

Size

  

Comments

  

  

  

/dev/sdb

  

DATA

  

8 GB

  

ASM Diskgroup forDATAFILES

  

  

  

/dev/sdc

  

FRA

  

8 GB

  

ASM Diskgroup for FLASH RECOVERY FILES

  

  

  

  
 

 

 

**Partition the Shared Disks:**

This section describes how to prepare the shared storage for Oracle RAC.

   

1.  Once the LUNs have been presented from the SAN to ALL servers in the cluster, partition the LUNs from one node only, run fdisk to create a single whole-disk partition with exactly 1 MB offset on each LUN to be used as ASM Disk

  
   

    [root@rac1 ~]# fdisk /dev/sdc

    Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel

    Building a new DOS disklabel. Changes will remain in memory only,

    until you decide to write them. After that, of course, the previous

    content won't be recoverable.

    Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

    Command (m for help): u

    Changing display/entry units to sectors

    Command (m for help): n

    Command action

    e   extended

    p   primary partition (1-4)

    p

    Partition number (1-4): 1

    First sector (63-2097151, default 63):

    Using default value 63

    Last sector or +size or +sizeM or +sizeK (63-2097151, default 2097151):

    Using default value 2097151

    Command (m for help): w

    The partition table has been altered!

    Calling ioctl() to re-read partition table.

    Syncing disks.

  
Load the updated block device partition tables by running the following on ALL servers participating in the cluster:

Repeate same for other disks.

**Creating udev rules(99-oracle-asmdevices.rules):**

Identify the Disks (/sbin/scsi_id)  

    # /sbin/scsi_id -g -u -s /block/sdb

  
SATA_VBOX_HARDDISK_VBd306dbe0-df3367e3\_  

    # /sbin/scsi_id -g -u -s /block/sdc

  
SATA_VBOX_HARDDISK_VB46dec7e0-192e8000\_

 

**Make SCSI Devices Trusted**

Add the following to the "/etc/scsi_id.config" file to configure SCSI devices as trusted. Create the file if it doesn't already exist.  

    options=-g

  
 

 

   

## Create UDEV Rules File:

  
Create the "/etc/udev/rules.d/99-oracle-asmdevices.rules" file.  

    # vi /etc/udev/rules.d/99-oracle-asmdevices.rules

    The file should contain the following lines for Oracle Linux 5. The PROGRAM parameter must match the command you used to retrieve the SCSI ID, and the RESULT parameter must match the value returned from your disks.

    KERNEL=="sd?1", BUS=="scsi", PROGRAM=="/sbin/scsi_id -g -u -s /block/$parent", RESULT=="SATA_VBOX_HARDDISK_VBd306dbe0-df3367e3_", NAME="asm-disk1", OWNER="oracle", GROUP="dba", MODE="0660"

    KERNEL=="sd?1", BUS=="scsi", PROGRAM=="/sbin/scsi_id -g -u -s /block/$parent", RESULT=="SATA_VBOX_HARDDISK_VB46dec7e0-192e8000_", NAME="asm-disk2", OWNER="oracle", GROUP="dba", MODE="0660"

  
 

**Load Updated Block Device Partitions (/sbin/partprobe)**

Load updated block device partition tables.  

    # /sbin/partprobe /dev/sdb1

    # /sbin/partprobe /dev/sdc1

  
 

**Test Rules (udevtest)**

Test the rules are working as expected.

\# \#OL5  

    # udevtest /block/sdb/sdb1

    # udevtest /block/sdc/sdc1

  
 

**Check Ownership and Permissions**

Check the disks are now available with the "asm-disk\*" alias and the correct ownership and permissions.

\# cd /dev

\# ls -al asm-disk\*  

    brw-rw---- 1 oracle dba 8, 17 Apr  8 22:47 asm-disk1

    brw-rw---- 1 oracle dba 8, 33 Apr  8 22:47 asm-disk2

  
 

remove the asm disk from vm and reattach after cloning

**clone rac2 node from rac1:**

shutdown the server  

    shutdown –h now

  
 

**clone  the virtual machine rac1 to rac2**

start the rac2 mechine

change the nework ips

system-config-network  

  

  

  

Active on Boot

  

Devices

  

IPV4.Netmask

  

IPV6/Prefix

  

  

  

Yes

  

eth0

  

192.168.1.12/255.255.255.0

  

Disable

  

  

  

Yes

  

eth1

  

192.168.2.12/255.255.255.0

  

Disable

  

  

  

  
 

rm –rf /etc/udev/rules.d/70-presistent-net.rules

 

sed –i –e   ‘/HWADDR/d’   -e  ‘/UUID/d’   /etc/sysconfig/network-scripts/ifcfg-eth\[0-1\]

Reboot

Ip l

Ls –l /dev/asm\*

Service named stop

 

**Step by step DNS Configuration on Linux RHEL5/OEL5**

Verify the IP address for the DNS server. Use following command to check the IP address.  

    [root@rac1 ~]# ifconfig

    eth0 Link encap:Ethernet HWaddr 00:0C:29:7D:60:F3

    inet addr:192.168.1.12 Bcast:192.168.1.255 Mask:255.255.255.0

    inet6 addr: fe80::20c:29ff:fe7d:60f3/64 Scope:Link

    UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1

    RX packets:28 errors:0 dropped:0 overruns:0 frame:0

    TX packets:39 errors:0 dropped:0 overruns:0 carrier:0

    collisions:0 txqueuelen:1000

    RX bytes:4919 (4.8 KiB) TX bytes:5628 (5.4 KiB)

    Interrupt:67 Base address:0x2024

  
There are four files we have to edit. Find the below easy steps to configure the DNS server.

Edit the named.conf file based on your configuration. For example.  

    options {

    listen-on port 53 { 192.168.1.12; };

    # listen-on-v6 port 53 { ::1; };

    directory "/var/named";

    dump-file "/var/named/data/cache_dump.db";

    statistics-file "/var/named/data/named_stats.txt";

    memstatistics-file "/var/named/data/named_mem_stats.txt";

    // Those options should be used carefully because they disable port

    // randomization

    // query-source port 53;

    // query-source-v6 port 53;

    allow-query { any; };

    allow-query-cache { localhost; };

    };

    logging {

    channel default_debug {

    file "data/named.run";

    severity dynamic;

    };

    };

    view localhost_resolver {

    match-clients { any; };

    match-destinations { 192.168.1.12; };

    recursion yes;

    include "/etc/named.rfc1912.zones";

    };

  
**Edit named.rfc1912.zones file. Sample file given below.**  

    zone "." IN {

    type hint;

    file "named.ca";

    };

    zone "oracle.com" IN {

    type slave;

    file "forward.zone";

        masters { 192.168.1.11; };

    };

    zone "localhost" IN {

    type master;

    file "localhost.zone";

    allow-update { none; };

    };

    zone "1.168.192.in-addr.arpa" IN {

                     type slave;

    file "reverse.zone";

              masters { 192.168.1.11; };

    };

  
Restart the named services using below statement.  

    service named restart

  
Verify the DNS server using below statements

   

    [root@dnc named]# dig rac1.oracle.com

    ; «» DiG 9.3.6-P1-RedHat-9.3.6-4.P1.el5_4.2 «» rac1.oracle.com

    ;; global options: printcmd

    ;; Got answer:

    ;; -»HEADER«- opcode: QUERY, status: NOERROR, id: 1483

    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 0

    ;; QUESTION SECTION:

    ;rac1.oracle.com. IN A

    ;; ANSWER SECTION:

    rac1.oracle.com. 86400 IN A 192.168.1.11

    ;; AUTHORITY SECTION:

    india.com. 86400 IN NS dnc.india.com.

    ;; Query time: 1 msec

    ;; SERVER: 192.168.1.11#53(192.168.1.11)

    ;; WHEN: Mon Aug 27 23:54:49 2012

    ;; MSG SIZE rcvd: 61

  
**Perform the same for the other**  

    dig rac2.oracle.com

    dig rac1-vip.oracle.com

    dig rac2-vip.oracle.com

    dig rac-scan.oracle.com

  
 

To check reverse the zone  

    [root@dnc named]# dig -x 192.168.1.11

    ; «» DiG 9.3.6-P1-RedHat-9.3.6-4.P1.el5_4.2 «» -x 192.168.1.11

    ;; global options: printcmd

    ;; Got answer:

    ;; -»HEADER«- opcode: QUERY, status: NOERROR, id: 55949

    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

    ;; QUESTION SECTION:

    ;11.1.168.192.in-addr.arpa. IN PTR

    ;; ANSWER SECTION:

    11.1.168.192.in-addr.arpa. 86400 IN PTR rac1.oracle.com.

    11.1.168.192.in-addr.arpa. 86400 IN PTR rac1.oracle.com.

    ;; AUTHORITY SECTION:

    1.168.192.in-addr.arpa. 86400 IN NS rac1.oracle.com.

    ;; ADDITIONAL SECTION:

    rac1.oracle.com. 86400 IN A 192.168.1.11

    ;; Query time: 1 msec

    ;; SERVER: 192.168.1.11#53(192.168.1.11)

    ;; WHEN: Mon Aug 27 23:57:27 2012

    ;; MSG SIZE rcvd: 124

  
**Perform the same with others**

   

13. **To verify DNS Server using nslookup**

  

    [root@dnc named]# nslookup rac1.oracle.com

    Server: 192.168.1.11

    Address: 192.168.1.11#53

    Name: rac1.oracle.com

    Address: 192.168.1.11

  
 

DNS server working fine now. Finally we got success.

 

Restart named service as a root user.  

    [root@dnc named]# service named restart

    Stopping named:                                            [  OK  ]

    Starting named:                                            [  OK  ]

  
 

Verify the SCAN names using nslookup.  

    [root@dnc named]# nslookup rac-scan

    Server:         192.168.1.11

    Address:        192.168.1.11#53

    Name:   rac-scan.oracle.com

    Address: 192.168.1.31

    Name:   rac-scan.oracle.com

    Address: 192.168.1.32

    Name:   rac-scan.oracle.com

    Address: 192.168.1.33

  
 

Now we are ready for installation.

 

**GRID INSTALLTION:**  

    su- oracle

    Download the software to /opt folder and unzip

    Unzip ‘*’

    cd grid

    cd rpm

    rpm –Uvh <rpm name>

    cd ..

    cd ssh<TAB>

    ./SshUsersetup.sh  -hosts “ rac1 rac2” –noPromptpassphrase –confirm -advanced

  
 

**Run prechecks**  

    ./runcluvfy.sh stage –pre crsinst –n  rac1,rac2  -fixup –verbose

  
 

./runInstaller

Skip updates  

- Install and configure grid infracture for cluster
- Typical installation
- Specity cluster configuration  
  - Scan name:rac-scan
  - Sshconnectivity: oracle password:XXXXXXX
  - Setup
  - 

    
- Create asm disk group  
  - Disk group name: data
  - Redundancy: external
  - Add disks:  
    - Candidate disks
    - /dev/asm-disk1

      
  - Next

    
- Create inventory à next
- Perform prerequisite checks à next
- Summay à next
- Run root scripts as root user
- Finish

  
**Installation of database:**  

- Configure security updates  
  - Uncheck box à next

    
- Select installation option:  
  - Create and configure a database

    
- System class  
  - Server class

    
- Node selection  
  - Real application cluster

    
- Select install type  
  - Typical install

    
- Typical install configuration  
  - Oracle base
  - Software location
  - Storage type: automatic storage management
  - Database file location: data
  - Asmsnmp password : XXXX
  - Osdba group: dba
  - Global database name:racdb

    
- Next
- Prechecks  
  - \>next

    
- Run root scripts.
