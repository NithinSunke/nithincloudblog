---
title: "How to solve “Device eth0 does not seem to be present, delaying initialization” error"
author: "Sunke Nithin"
date: 2019-07-11
categories: ["Linux"]
tags: []
source: https://nithinsunke.wordpress.com/2019/07/11/how-to-solve-device-eth0-does-not-seem-to-be-present-delaying-initialization-error/
---

# How to solve “Device eth0 does not seem to be present, delaying initialization” error

Today, I encountered with a strange error after I cloned oracle linux 6 guest machine in vmware. After logging in to oracle linux guest, my network card is disappeared. I can’t SSH to the guest mechine anymore. I tried to check the IP address using command:  

    # ifconfig

    lo Link encap:Local Loopback

    inet addr:127.0.0.1 Mask:255.0.0.0

    inet6 addr: ::1/128 Scope:Host

    UP LOOPBACK RUNNING MTU:65536 Metric:1

    RX packets:0 errors:0 dropped:0 overruns:0 frame:0

    TX packets:0 errors:0 dropped:0 overruns:0 carrier:0

    collisions:0 txqueuelen:0

    RX bytes:0 (0.0 b) TX bytes:0 (0.0 b)

  
eth0 config details doesn’t show up. It only displays the loopback interface details.I tried to restart the network service to sort out the issue.  

    # service network restart

  
Nothing! It simply displayed the following error.

Bringing up interface eth0: Device eth0 does not seem to be present, delaying initialization

I thought that there might be any misconfiguration in network configuration file. So, I decided to dig into the eth0 configuration file to make sure if everything is correct.  

    # vi /etc/sysconfig/network-scripts/ifcfg-eth0

    DEVICE=eth0

    BOOTPROTO=none

    DEVICE=eth0

    GATEWAY=192.168.1.1

    HWADDR=08:00:27:DC:33:3F

    IPADDR=192.168.1.150

    NETMASK=255.255.255.0

    NM_CONTROLLED=no

    ONBOOT=yes

    TYPE=Ethernet

    USERCTL=no

  
Everything was perfect.

Again, I tried to wake up Network card using command:  

    # ifup eth0

  
But, still no luck! It keeps showing the same error message as below.

Device eth0 does not seem to be present, delaying initialization  
I added and configured an additional virtual network interface card. I restarted network service and guest mechine several times. But, nothing helped. I keep getting the same error over and over.

After a bit searching over the Internet, I found the solution. MAC address was the problem! After cloned, oracle linux guest was still using the old machine’s MAC address in the Virtual machine settings.

If you ever faced the same issue, you fix it as described in the following method.

To fix this problem, simply remove the file /etc/udev/rules.d/70-persistent-net.rules and reboot your system.

\# rm /etc/udev/rules.d/70-persistent-net.rules

Now, you will see the network card is up.  
Have a Good day!!
