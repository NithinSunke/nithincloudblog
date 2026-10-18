---
title: "Create the local yum repository from DVD"
author: "Sunke Nithin"
date: 2020-10-29
categories: ["Linux"]
tags: []
source: https://nithinsunke.wordpress.com/2020/10/29/create-the-local-yum-repository-from-dvd/
---

# Create the local yum repository from DVD

1\) mount the dvd on to linux server  
2)create the directory on linux server to copy the dvd  
mkdir -p /mnt/oel6  
3) copy the dvd to /mnt/oel6  
cd /run/media/nsunke/OL-7.5\\ Server.x86_64/  
cp -Rv \* /mnt/oel6/.

4\) create a repo file in /etc/yum.repo.d  
cd /etc/yum.repos.d  
ls  
mv public-yum-ol7.repo /opt  
vi oel7.repo  
\[oel7\]  
name=oel7  
baseurl=file:///mnt/oel6  
enabled=1  
gpgcheck=0

\[root@sundv1 yum.repos.d\]# yum repolist  
Loaded plugins: langpacks, ulninfo  
oel7 \| 3.6 kB 00:00:00  
(1/2): oel7/group_gz \| 144 kB 00:00:00  
(2/2): oel7/primary_db \| 4.8 MB 00:00:00  
repo id repo name status  
oel7 oel7 5,081  
repolist: 5,081
