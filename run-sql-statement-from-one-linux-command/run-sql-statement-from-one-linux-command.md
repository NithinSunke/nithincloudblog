---
title: "Run SQL statement from one Linux command"
author: "Sunke Nithin"
date: 2019-09-06
categories: ["DATABASE"]
tags: []
source: https://nithinsunke.wordpress.com/2019/09/06/run-sql-statement-from-one-linux-command/
---

# Run SQL statement from one Linux command

In this session we will see how to execute  sql statement from operating system.

   

    [oracle@orcldb ~]$ echo "select * from v$version;"|sqlplus -s system/xxxxx

    BANNER

    --------------------------------------------------------------------------------

    CON_ID

    ----------

    Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

    0

    PL/SQL Release 12.2.0.1.0 - Production

    0

    CORE 12.2.0.1.0 Production

    0
