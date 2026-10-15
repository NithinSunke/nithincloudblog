---
title: "SSH Passwordless Login Using SSH Keygen in 5 Easy Steps on linux servers."
author: "Sunke Nithin"
date: 2020-11-20
categories: ["Linux"]
tags: []
source: https://nithinsunke.wordpress.com/2020/11/20/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps-on-linux-servers/
---

# SSH Passwordless Login Using SSH Keygen in 5 Easy Steps on linux servers.

SSH Client : 192.168.1.21 ( **sundv.sunketech.in** )  
SSH Remote Host : 192.168.1.20 ( **sunqa.sunketech.in** )

If you are dealing with a number of Linux remote servers, then SSH Password-less login is one of the best ways to automate tasks such as automatic backups with scripts, synchronization files using SCP, and remote command execution.

Step 1: Create Authentication SSH-Keygen Keys on – (192.168.1.20)  
First login into server 192.168.1.20 with user oracle and generate a pair of public keys using the following command.

\[oracle@sunqa .ssh\]\$ **ssh-keygen -t rsa**

Generating public/private rsa key pair.  
Enter file in which to save the key (/home/oracle/.ssh/id_rsa): \[Press enter key\]  
Created directory '/home/oracle/.ssh'.  
Enter passphrase (empty for no passphrase): \[Press enter key\]  
Enter same passphrase again: \[Press enter key\]  
Your identification has been saved in /home/oraccle/.ssh/id_rsa.  
Your public key has been saved in /home/oracle/.ssh/id_rsa.pub.  
The key fingerprint is:  
5f:ad:40:00:8a:d1:9b:99:b3:b0:f8:08:99:c3:ed:d3 oracle@sunqa.sunketech.in  
The key's randomart image is:  
+--\[ RSA 2048\]----+  
\| ..oooE.++\|  
\| o. o.o \|  
\| .. . \|  
\| o . . o\|  
\| S . . + \|  
\| . . . o\|  
\| . o o ..\|  
\| + + \|  
\| +. \|  
+-----------------+

Step 2: Create .ssh Directory on – 192.168.1.21  
Use SSH from server 192.168.1.20 to connect server 192.168.1.21 using oracle as a user and create .ssh directory under it, using the following command.

\[oracle@sunqa ~\]\$ **ssh oracle@192.168.1.21 mkdir -p .ssh**

The authenticity of host '192.168.1.21 (192.168.1.21)' can't be established.  
RSA key fingerprint is 45:0e:28:11:d6:81:62:16:04:3f:db:38:02:la:22:4e.  
Are you sure you want to continue connecting (yes/no)? yes  
Warning: Permanently added '192.168.1.21' (ECDSA) to the list of known hosts.  
oracle@192.168.1.21's password: \[Enter Your Password Here\]

Step 3: Upload Generated Public Keys to – 192.168.1.21  
Use SSH from server 192.168.1.20 and upload a new generated public key (id_rsa.pub) on server 192.168.1.21 under oralce‘s .ssh directory as a file name authorized_keys.

\[oracle@sunqa ~\]\$ **cat .ssh/id_rsa.pub \| ssh oracle@192.168.1.21 'cat \>\> .ssh/authorized_keys'**

oracle@192.168.1.21's password: \[Enter Your Password Here\]

Step 4: Set Permissions on – 192.168.1.21  
Due to different SSH versions on servers, we need to set permissions on .ssh directory and authorized_keys file.

\[oracle@sunqa ~\]\$ **ssh oracle@192.168.1.21 "chmod 700 .ssh; chmod 640 .ssh/authorized_keys"**

sheena@192.168.1.21's password: \[Enter Your Password Here\]

Step 5: Login from 192.168.1.20 to 192.168.1.21 Server without Password  
From now onwards you can log into 192.168.1.21 as oracle user from server 192.168.1.20 as oracle user without a password.

\[oracle@sunqa ~\]\$ **ssh oracle@192.168.1.21**
