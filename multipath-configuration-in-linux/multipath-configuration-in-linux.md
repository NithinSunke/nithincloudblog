---
title: "Multipath configuration in linux"
author: "Sunke Nithin"
date: 2019-07-12
categories: ["Linux"]
tags: []
source: https://nithinsunke.wordpress.com/2019/07/12/multipath-configuration-in-linux/
---

# Multipath configuration in linux

• Install device-mapper-multipath package
    # yum install device-mapper-multipath.x86_64

    Copy multipath.conf file to /etc
    cp /usr/share/doc/device-mapper-multipath-0.4.9/multipath.conf /etc

*Execute command below command to obtain Vendor, Model and Port, Channel, SCSI-ID, LUN*

    [root@asmdb ~]# cat /proc/scsi/scsi
    Attached devices:
    Host: scsi4 Channel: 00 Id: 00 Lun: 00 Vendor: OPNFILER Model: VIRTUAL-DISK Rev: 0 Type: Direct-Access ANSI SCSI revision: 04

    Host: scsi3 Channel: 00 Id: 00 Lun: 00 Vendor: OPNFILER Model: VIRTUAL-DISK Rev: 0 Type: Direct-Access ANSI SCSI revision: 04

    Host: scsi5 Channel: 00 Id: 00 Lun: 00 Vendor: OPNFILER Model: VIRTUAL-DISK Rev: 0 Type: Direct-Access ANSI SCSI revision: 04

  
*To get WWID of LUN you can use the /dev/disk/by-id/ file:*  

    [root@asmdb ~]# ls -la /dev/disk/by-id/

    total 0

    lrwxrwxrwx. 1 root root 10 Sep 14 06:38 lvm-pv-uuid-r1cjbd-VqwE-rGgi-JFoqeBGf-UBEq-HAZOcC -> ../../sda2

    lrwxrwxrwx. 1 root root 9 Sep 14 06:39 scsi14f504e46494c450036386a5368642d543257722d46525562 -> ../../sde

    lrwxrwxrwx. 1 root root 10 Sep 14 06:39 scsi14f504e46494c450036386a5368642d543257722d46525562-part1 -> ../../sde1

    lrwxrwxrwx. 1 root root 9 Sep 14 06:39 scsi14f504e46494c450049774e6875512d326743342d76397558 -> ../../sdf

    lrwxrwxrwx. 1 root root 10 Sep 14 06:39 scsi14f504e46494c450049774e6875512d326743342d76397558-part1 -> ../../sdf1

    lrwxrwxrwx. 1 root root 9 Sep 14 06:39 scsi14f504e46494c450066764448386b2d39584f4b2d6e6d6f69 -> ../../sdd

    lrwxrwxrwx. 1 root root 10 Sep 14 06:39 scsi14f504e46494c450066764448386b2d39584f4b2d6e6d6f69-part1 -> ../../sdd1

    The wwid is 14f504e46494c450066764448386b2d39584f4b2d6e6d6f69

    Edit /etc/multipath.conf file and add the below entries

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

  
*Restart multipathd and configure daemon to startup on reboot*  

    # service multipathd restart

    Stopping multipathd daemon: [ OK ]

    Starting multipathd daemon: [ OK ]

  

    # chkconfig multipathd on

  
*Verify current multipath configuration*  

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

  
*Verify LUNS*  

    [root@asmdb dev]# cd /dev/mapper/

    [root@asmdb mapper]# ls -ltr

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
