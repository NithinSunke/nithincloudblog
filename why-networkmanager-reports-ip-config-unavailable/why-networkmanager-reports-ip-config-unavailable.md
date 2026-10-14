---
title: "Why NetworkManager reports ip-config-unavailable?"
author: "Sunke Nithin"
date: 2024-05-20
categories: ["Linux"]
tags: []
source: https://nithinsunke.wordpress.com/2024/05/20/why-networkmanager-reports-ip-config-unavailable/
---

# Why NetworkManager reports ip-config-unavailable?

Today when i trying to deploy oracle linux 7 vm using terrform on my lab environment i was hit by bug which stopping the network device eth0. the below steps from RHEL was the solution.

## Environment

- RedHat Enterprise Linux(7 and above)
- NetworkManager

## Issue

- Why NetworkManager reports ip-config-unavailable?
- Error shows while adding ip address:

``` wp-block-code
May 14 09:11:14 servera NetworkManager[675]: <info>  (ens8): device state change: ip-config -> secondaries (reason 'ip-config-unavailable') [70 90 5]

 May 14 09:03:27 serverb NetworkManager[695]: <info>  [1684047807.5263] device (ens9): state change: ip-config -> failed (reason 'ip-config-unavailable', sys-iface-state: 'managed')
```

## Resolution

- Follow the below steps to change the `ipv4.method` as `auto` or assign the nic for `static IPaddress`.After, bringing up the connection.

Method 1:

``` wp-block-code
nmcli connection modify <con-name> ipv4.method auto
nmcli connection up <con-name>
```

Method 2:

``` wp-block-code
nmcli connection modify <con-name> ipv4.method manual ipv4.address <IP>/subnet 
nmcli connection up <con-name>
```

## Root Cause

- The `ipv4.method` is disabled for the connection hence `no ipv4 configuration is available` for the interface. This restricts the access to add the IP address. When changing the ipv4.method to `auto` or `manual`, then onwards it allows making changes in the IP address for the interface.
