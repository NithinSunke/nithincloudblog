---
title: "Steps to install PostgreSQL in oel7"
author: "Sunke Nithin"
date: 2022-02-05
categories: ["PostgreSql"]
tags: []
source: https://nithinsunke.wordpress.com/2022/02/05/steps-to-install-postgresql-in-oel7/
---

# Steps to install PostgreSQL in oel7

In this blog post, we will see how to install PostgreSQL in Oracle Linux 7

Go to the site <https://www.postgresql.org/download/linux/redhat/>

Select the appropriate version and operating system

![](images/image.png)

Copy, paste and run the relevant parts of the setup script

sudo yum install -y <https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm>

![](images/image-2-1.jpeg)

sudo yum install -y postgresql12-server

![](images/image-2-2.jpeg)

![](images/image-2.jpeg)

![](images/image-1.jpeg)

sudo /usr/pgsql-12/bin/postgresql-12-setup initdb

sudo systemctl enable postgresql-12

![](images/image.jpeg)

sudo systemctl start postgresql-12

![](images/image-3.jpeg)

login as Postgres user and connect to the database and list the database

![](images/image-1.png)
