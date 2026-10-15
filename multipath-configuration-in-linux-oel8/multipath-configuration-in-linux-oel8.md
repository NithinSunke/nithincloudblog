---
title: "Multipath Configuration in Linux (OEL8)"
author: "Sunke Nithin"
date: 2025-04-08
categories: ["Linux"]
tags: ["iscsi", "multipath", "san", "san", "storage", "tools"]
source: https://nithinsunke.wordpress.com/2025/04/08/multipath-configuration-in-linux-oel8/
---

# Multipath Configuration in Linux (OEL8)

When working with enterprise-grade storage systems like SAN (Storage Area Network), **multipath I/O (MPIO)** is a must-have for redundancy and performance. It allows your Linux system to access the same storage device through multiple physical paths — ensuring **high availability and failover**.

In this post, we'll configure **device-mapper multipath** on **Oracle Linux 8 (OEL8)** and connect it to shared SAN storage.

## 🧠 What is Multipath?

Multipath I/O enables multiple physical paths between the server and storage device. If one path fails, I/O is seamlessly routed through another path without disrupting services.

**Benefits:**

- ✅ Redundancy
- ✅ Load balancing
- ✅ High availability
- ✅ Better throughput

## 🧰 Prerequisites

- Oracle Linux 8 installed
- SAN Storage connected (iSCSI)
- At least 1 paths from the server to the same LUN( as resource constriants am using only one path)
- Root access to the system

### ⚙️ Step-by-Step Multipath Setup

Install Required Packages

``` wp-block-code
# yum install device-mapper-multipath.x86_64
```

Copy multipath.conf file to /etc

``` wp-block-code
cp /usr/share/doc/device-mapper-multipath-0.4.9/multipath.conf /etc
```

E*xecute command below command to obtain Vendor, Model and Port, Channel, SCSI-ID, LUN*

``` wp-block-code
[root@asmdb ~]# cat /proc/scsi/scsi
Attached devices:
Host: scsi4 Channel: 00 Id: 00 Lun: 00 Vendor: OPNFILER Model: VIRTUAL-DISK Rev: 0 Type: Direct-Access ANSI SCSI revision: 04

Host: scsi3 Channel: 00 Id: 00 Lun: 00 Vendor: OPNFILER Model: VIRTUAL-DISK Rev: 0 Type: Direct-Access ANSI SCSI revision: 04

Host: scsi5 Channel: 00 Id: 00 Lun: 00 Vendor: OPNFILER Model: VIRTUAL-DISK Rev: 0 Type: Direct-Access ANSI SCSI revision: 04
```

*To get WWID of LUN you can use the /dev/disk/by-id/ file:*

``` wp-block-code
[root@asmdb ~]#ls -la /dev/disk/by-id/

total 0

lrwxrwxrwx. 1 root root 10 Sep 14 06:38 lvm-pv-uuid-r1cjbd-VqwE-rGgi-JFoqeBGf-UBEq-HAZOcC -> ../../sda2

lrwxrwxrwx. 1 root root 9 Sep 14 06:39 scsi14f504e46494c450036386a5368642d543257722d46525562 -> ../../sde
lrwxrwxrwx. 1 root root 10 Sep 14 06:39 scsi14f504e46494c450036386a5368642d543257722d46525562-part1 -> ../../sde1
lrwxrwxrwx. 1 root root 9 Sep 14 06:39 scsi14f504e46494c450049774e6875512d326743342d76397558 -> ../../sdf
lrwxrwxrwx. 1 root root 10 Sep 14 06:39 scsi14f504e46494c450049774e6875512d326743342d76397558-part1 -> ../../sdf1
lrwxrwxrwx. 1 root root 9 Sep 14 06:39 scsi14f504e46494c450066764448386b2d39584f4b2d6e6d6f69 -> ../../sdd
lrwxrwxrwx. 1 root root 10 Sep 14 06:39 scsi14f504e46494c450066764448386b2d39584f4b2d6e6d6f69-part1 -> ../../sdd1

The wwid is 14f504e46494c450066764448386b2d39584f4b2d6e6d6f69Edit /etc/multipath.conf file and add the below entries

defaults {
udev_dir /dev
polling_interval 10
path_selector "round-robin 0"
path_grouping_policy multibus
getuid_callout "/lib/udev/scsi_id --whitelisted --device=/dev/%n"
# prio alua
path_checker readsector0
rr_min_io 100
max_fds 8192
rr_weight priorities
failback immediate
no_path_retry fail
user_friendly_names yes }

blacklist {
# Blacklist by WWID
wwid "*"
}

blacklist_exceptions {
wwid "14f504e46494c450036386a5368642d543257722d46525562"
wwid "14f504e46494c450066764448386b2d39584f4b2d6e6d6f69"
wwid "14f504e46494c450049774e6875512d326743342d76397558"
}

multipaths {
multipath {
wwid 14f504e46494c450036386a5368642d543257722d46525562
alias ocr
}
multipath {
wwid 14f504e46494c450066764448386b2d39584f4b2d6e6d6f69
alias data
}
multipath {
wwid 14f504e46494c450049774e6875512d326743342d76397558
alias fra
}
```

*Restart multipathd and configure daemon to startup on reboot*

``` wp-block-code
# service multipathd restart
Stopping multipathd daemon: [ OK ]
Starting multipathd daemon: [ OK ]

# chkconfig multipathd on
```

*Verify current multipath configuration*

``` wp-block-code
# multipath –ll
Sep 14 06:52:52 | /etc/multipath.conf line 98, duplicate keyword: defaults
fra (14f504e46494c450049774e6875512d326743342d76397558) dm-4 OPNFILER,VIRTUAL-DISK
size=3.9G features='0' hwhandler='0' wp=rw
`-+- policy='round-robin 0' prio=1 status=active
`- 5:0:0:0 sdf 8:80 active ready running
data (14f504e46494c450066764448386b2d39584f4b2d6e6d6f69) dm-2 OPNFILER,VIRTUAL-DISK
size=3.9G features='0' hwhandler='0' wp=rw
`-+- policy='round-robin 0' prio=1 status=active
`- 4:0:0:0 sdd 8:48 active ready running
ocr (14f504e46494c450036386a5368642d543257722d46525562) dm-3 OPNFILER,VIRTUAL-DISK
size=4.0G features='0' hwhandler='0' wp=rw
`-+- policy='round-robin 0' prio=1 status=active
`- 3:0:0:0 sde 8:64 active ready running
```

*Verify LUNS*

``` wp-block-code
[root@asmdb dev]#cd /dev/mapper/
[root@asmdb mapper]#ls -ltr
total 0
crw-rw----. 1 root root 10, 236 Sep 14 06:38 control
lrwxrwxrwx. 1 root root 7 Sep 14 06:38 vg_linux-lv_swap -> ../dm-1
lrwxrwxrwx. 1 root root 7 Sep 14 06:38 vg_linux-lv_root -> ../dm-0
lrwxrwxrwx. 1 root root 7 Sep 14 06:52 data -> ../dm-2
lrwxrwxrwx. 1 root root 7 Sep 14 06:52 fra -> ../dm-4
lrwxrwxrwx. 1 root root 7 Sep 14 06:52 datap1 -> ../dm-6
lrwxrwxrwx. 1 root root 7 Sep 14 06:52 ocr -> ../dm-3
lrwxrwxrwx. 1 root root 7 Sep 14 06:52 frap1 -> ../dm-7
lrwxrwxrwx. 1 root root 7 Sep 14 06:52 ocrp1 -> ../dm-5
```

## 🧩 Conclusion

Multipath is an essential feature in any enterprise storage setup. With just a few steps, your OEL8 server can enjoy enhanced **availability, redundancy, and performance** by leveraging multiple paths to storage.

Whether you're running databases, virtual machines, or file shares — multipath ensures your data stays accessible even when hardware fails.

## 📜 Disclaimer

This guide is intended for educational and lab purposes. Always validate configurations in a test environment before deploying in production. The author is not responsible for any data loss or system misconfiguration.
