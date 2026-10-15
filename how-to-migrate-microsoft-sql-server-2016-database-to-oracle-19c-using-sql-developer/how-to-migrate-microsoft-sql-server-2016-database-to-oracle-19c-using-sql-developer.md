---
title: "How to Migrate Microsoft SQL Server 2016 Database to Oracle 19c Using SQL Developer"
author: "Sunke Nithin"
date: 2025-04-11
categories: ["MS SQLSERVER", "ORACLE"]
tags: ["DATABASE", "SQL", "sql-server", "technology"]
source: https://nithinsunke.wordpress.com/2025/04/11/how-to-migrate-microsoft-sql-server-2016-database-to-oracle-19c-using-sql-developer/
---

# How to Migrate Microsoft SQL Server 2016 Database to Oracle 19c Using SQL Developer

Migrating from SQL Server to Oracle might seem complex, but Oracle SQL Developer provides an integrated, GUI-based method to handle both schema conversion and data transfer.

### ✅ Prerequisites

Before beginning, ensure you have:

| Component          | Description                               |
|--------------------|-------------------------------------------|
| 🖥️ SQL Server 2016 | Source database                           |
| 🛢 Oracle 19c       | Target database                           |
| 🧰 SQL Developer   | Installed on your system (latest version) |
| 🔌 JDBC Driver     | SQL Server JDBC (`jtds.jar`)              |
| 🔑 DB Credentials  | Admin or migration-level privileges       |

Before starting migration create a sample table

here i am creating a table with column that contain images as blob column.

![](images/image-274.png)

![](images/image-276.png)

i have added 11 rows to the table.

### 🛠 Step 1: Setup SQL Developer

1.  **Download & Install SQL Developer**:  
    [Download link](https://www.oracle.com/tools/downloads/sqldev-downloads.html)
2.  **Add the SQL Server JDBC Driver**:
    - Go to `Tools > Preferences > Database > Third Party JDBC Drivers`

![](images/image-224.png)

3\. Click **Add Entry**, select `jtds.jar`

download jtds sqlserver jdbc driver

![](images/image-226.png)

![](images/image-228.png)

4\. Restart SQL Developer

then you will able to see sql server database connection type.

![](images/image-230.png)

![](images/image-232.png)

once you entered hostname and port no. hit on retrieve database then you will find all the associated databases to the instance. select the one you want to migrate and test the connection and save/connect.

![](images/image-234.png)

Create a migration Repository to hold the migration related data.

![](images/image-237.png)

![](images/image-239.png)

![](images/image-241.png)

![](images/image-243.png)

select the database that you want to migrate and choose the option migrate to oracle.

![](images/image-244.png)

![](images/image-245.png)

select the migration repository schema

![](images/image-246.png)

at this step provide name to the project and provide the file system directory to hold the migration files.

![](images/image-248.png)

here select the 3rd party database our case ms sqlserver.

![](images/image-250.png)

![](images/image-252.png)

in this screen it will show how datatype convertion:

![](images/image-253.png)

![](images/image-255.png)

here in the step of target database where the user is admin user in my case i have used system user.

![](images/image-256.png)

shows the source and target database connection details:

![](images/image-258.png)

![](images/image-260.png)

![](images/image-262.png)

migration is running and completed successfully

![](images/image-263.png)

from the migration tab you can see more details about the migration job.

![](images/image-264.png)

login in to oracle database you can find a schema with db name

![](images/image-266.png)

![](images/image-268.png)

edit and unlock the user and reset the password

![](images/image-270.png)

![](images/image-272.png)

create connection to new migrated schema

![](images/image-278.png)

validate the data and objects:

![](images/image-280.png)

it has 11 rows so all the rows are matched.

Cleanup and Optimize

Gather statistics:

``` wp-block-code
EXEC DBMS_STATS.GATHER_SCHEMA_STATS('IMAGEDB');
```

## 📌 Tips

- Run migration in a test environment first
- Avoid migrating system tables or unsupported data types
- For large databases, consider breaking the migration into parts

## ⚠️ Disclaimer

**This post is for educational purposes only.** Ensure you have the proper **licenses and approvals** before migrating databases in a production environment. Always perform migrations in a **test environment** first. Oracle and Microsoft trademarks and product names used here belong to their respective owners.

>
