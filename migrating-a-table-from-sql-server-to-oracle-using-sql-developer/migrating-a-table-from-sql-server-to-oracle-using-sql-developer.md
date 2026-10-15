---
title: "Migrating a Table from SQL Server to Oracle Using SQL Developer"
author: "Sunke Nithin"
date: 2025-04-11
categories: ["MS SQLSERVER", "ORACLE"]
tags: ["AZURE", "CLOUD", "DATABASE", "SQL"]
source: https://nithinsunke.wordpress.com/2025/04/11/migrating-a-table-from-sql-server-to-oracle-using-sql-developer/
---

# Migrating a Table from SQL Server to Oracle Using SQL Developer

Moving from Microsoft SQL Server to Oracle? Oracle SQL Developer provides a built-in migration tool that helps you migrate your tables, data, and even PL/SQL code with ease.

In this blog post, we’ll walk through **how to migrate a table from SQL Server to Oracle** using SQL Developer — step-by-step, with screenshots

**🧰** **Prerequisites**

Before we get started, make sure you have:

- ✅ Oracle SQL Developer installed ([Download here](https://www.oracle.com/tools/downloads/sqldev-downloads.html))
- ✅ Microsoft SQL Server running with a database you want to migrate
- ✅ Oracle Database installed and accessible
- ✅ SQL Server JDBC driver (e.g., sqljdbc.jar)

I Have download and installed latest sql developer in windows pc.

![](images/image-172.png)

source Microsoft SQL Server 2016 is running with a database that want to migrate

![](images/image-174.png)

Oracle Database 19c installed and accessible

![](images/image-176.png)

NCDB is the pluggable database and imadm is the schema in oracle database where we are going to migrate the table.

At **source database** ms sqlserver database I have create a database and table that hold images and binary object

![](images/image-178.png)

![](images/image-180.png)

**Add SQL Server JDBC Driver to SQL Developer**

1.  `Download jtds sql server jdbc driver`

![](images/image-182.png)

2\. Open **SQL Developer**

3\. Navigate to `Tools > Preferences`

![](images/image-184.png)

4\. Go to `Database > Third Party JDBC Drivers`

![](images/image-186.png)

5\. Click **Add Entry** and select the `sqljdbc.jar` file you downloaded

![](images/image-188.png)

6\. Click **OK** and restart SQL Developer

Once the sql developer is restarted you will see the ms sqlserver database connection type

![](images/image-190.png)

Creating sql server database connection on sql developer

![](images/image-192.png)

hostname of ms sqlserver and tcp/ip port 1433

**Note: Pre-reqs for the connection to get success**

1.  In sql server configuration manager under sql server network configuration protocols for sqldb tcp/ip status should be enable and make note of port number it is configured.

![](images/image-194.png)

![](images/image-196.png)

**Create a SQL Server Connection:**

In sqldeveloper update the details as below and test connection

![](images/image-198.png)

**Create a Migration Repository:**

In oracle database create a migration schema

Schema name: mig_repo_schema

Privileges: CONNECT,RESOURCE,CREATE SESSION,CREATE VIEW,CREATE              MATERIALIZED VIEW

``` wp-block-code
GRANT CONNECT, RESOURCE, CREATE SESSION, CREATE VIEW, CREATE MATERIALIZED VIEW TO mig_repo_schema;
```

SQL Developer stores migration metadata in a repository.

1.  Go to `Tools > Migration > Repository Management > `Select `Create Repository`

![](images/image-200.png)

2\. Choose your Oracle connection and follow the wizard to create the repository

![](images/image-202.png)

![](images/image-204.png)

![](images/image-206.png)

**Start the Migration Wizard:**

From sql server connection select the database and select the table that you want to migrate

![](images/image-208.png)

![](images/image-210.png)

![](images/image-214.png)

Migration is completed login into target schema in oracle database and validate:

![](images/image-216.png)

Source and target have same number of rows.

## Tips

-  Always take a backup before migration
-  Migrate smaller tables first to validate the process
-  Use staging schemas to test before moving to production

## Conclusion

Migrating from SQL Server to Oracle might sound intimidating, but with Oracle SQL Developer’s powerful migration wizard, it becomes a manageable and guided process.

This tool not only helps convert the structure but also handles data migration with minimal hassle — making it a great option for DBAs and developers alike.

## 📜 Disclaimer

This guide is intended for educational and lab purposes. Always validate configurations in a test environment before deploying in production. The author is not responsible for any data loss or system misconfiguration.
