---
title: "How to Configure Duo MFA on Linux (SSH) – Step-by-Step"
author: "Sunke Nithin"
date: 2025-04-08
categories: ["Linux", "ORACLE"]
tags: ["cybersecurity", "devops", "security", "technology"]
source: https://nithinsunke.wordpress.com/2025/04/08/how-to-configure-duo-mfa-on-linux-ssh-step-by-step/
---

# How to Configure Duo MFA on Linux (SSH) – Step-by-Step

Enabling **multi-factor authentication** (MFA) on your Linux server is one of the best security upgrades you can make, especially if you're managing sensitive workloads or remote systems. Duo Security provides a solid and straightforward solution to implement MFA for SSH access.

In this blog post, we'll cover the full setup—from Duo Admin Panel to your Linux server. Let's dive in

## 🛠️ Prerequisites

Before starting, ensure you have:

- A Linux server (tested on Ubuntu, CentOS, Oracle Linux, etc.)
- SSH access
- Root or sudo privileges
- A [Duo Security](https://duo.com) account (free for up to 10 users)
- A smartphone with Duo Mobile app installed

## 📘 Step 1: Create a Duo Application

1.  Log in to the Duo Admin Panel.
2.  Go to **Applications** → **Protect an Application**.
3.  Search for **Unix Application** and click **Protect**.
4.  Note down the following:
    - **Integration Key**
    - **Secret Key**
    - **API Hostname**

Keep these safe—we’ll need them for configuration.

## Step 2: Install Duo Unix Package on Linux

``` wp-block-code
yum install openssl-devel -y

yum install pam-devel -y

yum install bzip2 -y
```

Create /etc/yum.repos.d/duosecurity.repo with the following contents:

``` wp-block-code
[duosecurity]
name=Duo Security Repository
baseurl=https://pkg.duosecurity.com/RedHat/$releasever/$basearch
enabled=1
gpgcheck=1
```

Execute the following shell commands for Red Hat 6 and later:

``` wp-block-code
rpm --import https://duo.com/DUO-GPG-PUBLIC-KEY.asc
yum install duo_unix -y
```

## Step 3: Integrate Duo with SSH (PAM)

edit /etc/pam.d/password-auth and /etc/pam.d/system-auth

update the line

``` wp-block-code
auth        requisite    pam_unix.so nullok try_first_pass
auth        sufficient   /lib64/security/pam_duo.so
```

``` wp-block-code
[root@o18cdb01 ~]# getsebool -a|grep duo
pam_duo_permit_local_login --> off
pam_duo_permit_sshd --> on
[root@o18cdb01 ~]# setsebool pam_duo_permit_local_login on
[root@o18cdb01 ~]# getsebool -a|grep duo
pam_duo_permit_local_login --> on
pam_duo_permit_sshd --> on
[root@o18cdb01 ~]#
```

## Step 4: Configure Duo

update the below file /etc/duo/pam_duo.conf

``` wp-block-code
[root@o18cdb01 ~]# cat /etc/duo/pam_duo.conf

[duo]
; Duo integration key
ikey = <integration key>
; Duo secret key
skey = <secret key>
; Duo API host
host = api-xxxxxxxx.duosecurity.com
; `failmode = safe` In the event of errors with this configuration file or connection to the Duo service
; this mode will allow login without 2FA.
; `failmode = secure` This mode will deny access in the above cases. Misconfigurations with this setting
; enabled may result in you being locked out of your system.
failmode = safe
; Send command for Duo Push authentication
pushinfo = yes
autopush = yes
```

## 📲 Step 5: Test Duo MFA

1.  Open a new terminal and SSH into the server.
2.  After entering your password, you'll receive a Duo push or prompt.
3.  Approve it on your phone to complete login.

``` wp-block-code
[root@o18cdb01 ~]# su - nsunke
Last login: Sun Mar  9 11:27:10 UTC 2025 from 172.16.0.100 on pts/0
[nsunke@o18cdb01 ~]$ su - nsunke
Password:
Autopushing login request to phone...

Pushed a login request to your device...
Success. Logging you in...
Last login: Sun Mar  9 12:44:44 UTC 2025 on pts/0
[nsunke@o18cdb01 ~]$
```

## ✅ Wrapping Up

You've now successfully secured your Linux server using **Duo MFA**! 🎉 This adds an essential second layer of security, especially critical for cloud servers or remote access systems.

## **Disclaimer:**

The information provided in this blog post is for educational and informational purposes only. While we strive to ensure the accuracy and timeliness of the content, we make no guarantees regarding its completeness or applicability to your specific situation. Implementing security measures such as Duo Multi-Factor Authentication (MFA) involves risks, and it's recommended to consult with a qualified IT professional before making any changes to your system. We disclaim any liability for any loss or damage resulting from the use of the information provided herein.
