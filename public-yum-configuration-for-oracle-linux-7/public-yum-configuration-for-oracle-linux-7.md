---
title: "PUBLIC YUM CONFIGURATION FOR ORACLE LINUX 7"
author: "Sunke Nithin"
date: 2019-12-17
categories: ["Linux"]
tags: []
source: https://nithinsunke.wordpress.com/2019/12/17/public-yum-configuration-for-oracle-linux-7/
---

# PUBLIC YUM CONFIGURATION FOR ORACLE LINUX 7

A quick setup of configuring yum  on oracle linux 6  to use public oracle yum server.  

1.  Login as root user.
2.  Go to /etc/yum.repos.d

  
**             cd /etc/yum.repos.d**  

3.  Execute the below command.

  
**            wget http://yum.oracle.com/public-yum-ol7.repo**  

4.  It will create a file called **public-yum-ol6.repo.**

  
Edit the file and enable other repositories by setting **enable=1** from 0.  

5.  Run yum repolist to show the registered channels and you see everything including the latest published RPMs.
6.  Now you can run yum update to update.
