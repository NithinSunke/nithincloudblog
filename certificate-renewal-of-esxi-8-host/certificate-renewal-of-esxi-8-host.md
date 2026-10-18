---
title: "certificate renewal of esxi 8 host"
author: "Sunke Nithin"
date: 2024-08-02
categories: ["CLOUD", "LAB"]
tags: []
source: https://nithinsunke.wordpress.com/2024/08/02/certificate-renewal-of-esxi-8-host/
---

# certificate renewal of esxi 8 host

in the blog i will brief the steps to renewal of ssl certificate on esxi host

**on xca**  
generate csr file  
exprot pem chain  
export private key

**on esxi host**

cd /etc/vmware/ssl/  
\[root@vesxi01:/etc/vmware/ssl\] cp rui.crt rui.crt_org  
\[root@vesxi01:/etc/vmware/ssl\] cp rui.key rui.key_org

copy the cert files to server  
\[root@vesxi01:/etc/vmware/ssl\] cp esxi01.scslabs.xyz.key rui.key  
cp: overwrite 'rui.key'? y  
\[root@vesxi01:/etc/vmware/ssl\] cp esxi01.scslabs.xyz.crt rui.crt  
cp: overwrite 'rui.crt'? y

restart the hostd

\[root@vesxi01:/etc/vmware/ssl\] /etc/init.d/hostd restart  
hostd stopped.  
hostd started.
