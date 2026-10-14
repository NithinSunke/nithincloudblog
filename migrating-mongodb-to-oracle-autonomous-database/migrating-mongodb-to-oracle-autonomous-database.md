---
title: "Migrating MongoDB to Oracle Autonomous Database"
author: "Sunke Nithin"
date: 2025-09-01
categories: ["MongoDB"]
tags: []
source: https://nithinsunke.wordpress.com/2025/09/01/migrating-mongodb-to-oracle-autonomous-database/
---

# Migrating MongoDB to Oracle Autonomous Database

In today's data-driven world, businesses often need to transition between database systems to leverage advanced features like SQL analytics, AI capabilities, and scalable cloud infrastructure. If you're using MongoDB and considering a move to Oracle Autonomous Database, the process is surprisingly straightforward. Oracle's MongoDB API compatibility allows you to use familiar MongoDB tools while unlocking the power of Oracle's SQL and AI features. This blog post, inspired by the Oracle MongoDB API documentation, guides you through migrating MongoDB data to Oracle Autonomous Database and creating SQL views for advanced JSON data analysis.

## Why Migrate to Oracle Autonomous Database?

Oracle Autonomous Database offers a fully managed cloud database that supports both JSON and relational workloads. With its MongoDB API compatibility, you can:

- Continue using MongoDB tools like Mongo Compass and mongosh.
- Seamlessly query JSON data using SQL for relational-style analysis.
- Leverage Oracle Database 23ai features like AI Vector Search, Retrieval-Augmented Generation (RAG), and JSON Relational Duality.

This makes it an ideal choice for organizations looking to modernize their data infrastructure without abandoning existing MongoDB workflows.

## Step 1: Setting Up Your Oracle Autonomous Database

To begin, create an Oracle Autonomous Database instance configured for JSON or Transaction Processing workloads. Here's how:

1.  **Log in to Oracle Cloud Infrastructure (OCI)**: Access your OCI account. If you don’t have one, sign up for a free account with Oracle’s Always Free Resources.
2.  **Create an Autonomous Database**:
    - Navigate to the Autonomous Database page in the OCI console.
    - Click **Create Autonomous Database**.
    - Set the **Display name** and **Database name** (e.g., AJD1).
    - Choose **JSON** or **Transaction Processing** as the workload type.
    - Select database version **23ai** for the latest features.
    - Set a password for the ADMIN user and configure secure access for your IP.
    - Click **Create**.

Once created, retrieve the **Public access URL** for MongoDB API connectivity from the **Tool configuration** tab in the database details.

![](images/image-1.png)

## Step 2: Connecting with MongoDB Tools

Oracle’s MongoDB API allows you to connect using familiar tools like Mongo Compass or mongosh. Here’s an example using mongosh:

``` wp-block-code
mongosh 'mongodb://ADMIN:xxxxxxx@xixezm8d.adb.me-jeddah-1.oraclecloudapps.com:27017/ADMIN?authMechanism=PLAIN&authSource=$external&ssl=true&retryWrites=false&loadBalanced=true'
```

![](images/image-3.png)

Replace \<your-password\> and \<hostname-and-DB-URL\> with your database credentials and URL. This command establishes a connection, allowing you to interact with your Oracle database as if it were MongoDB.

Create a normal user in ATP database and grant below privilleges

![](images/image-5.png)

``` wp-block-code
GRANT CREATE SESSION, CREATE TABLE TO MONGOADM;
GRANT SODA_APP TO MONGOADM;
ALTER USER MONGOADM QUOTA UNLIMITED ON DATA;
```

``` wp-block-code
mongosh 'mongodb://mongoadm:xxxxx@xixezm8d.adb.me-jeddah-1.oraclecloudapps.com:27017/mongoadm?authMechanism=PLAIN&authSource=$external&ssl=true&retryWrites=false&loadBalanced=true'
```

![](images/image-7.png)

### Migrate your data using Mongo Shell

**At Source Database**

![](images/image-9.png)

In this blog we will migrate **sample_airbnb** database.

![](images/image-11.png)

export the collection using mongoexport

``` wp-block-code
mongoexport --db sample_airbnb --collection listingsAndReviews --out /opt/listingsAndReviews.json --username mongoadmin --authenticationDatabase admin
```

![](images/image-13.png)

**Connect to target database ATP:**

![](images/image-15.png)

**import the backup json file to target ATP database**

``` wp-block-code
mongoimport 'mongodb://mongoadm@xixezm8d.adb.me-jeddah-1.oraclecloudapps.com:27017/mongoadm?authMechanism=PLAIN&authSource=$external&ssl=true&retryWrites=false&loadBalanced=true' --collection listingsAndReviews /opt/listingsAndReviews.json
```

![](images/image-17.png)

List the Collections

![](images/image-19.png)

List the row count at ATP db.

![](images/image-21.png)

List the data using JSON_VALUE to define columns.

![](images/image-23.png)

## Leveraging Oracle’s Advanced Features

With your JSON data in Oracle Autonomous Database, you can take advantage of powerful features:

- **AI Vector Search**: Enable semantic search on your JSON data.
- **JSON Relational Duality**: Seamlessly switch between JSON and relational representations.
- **Select AI**: Use natural language to query your data.

These capabilities make your data “AI-ready,” opening up new possibilities for analytics and application development.

## Conclusion

Migrating from MongoDB to Oracle Autonomous Database is a seamless process that retains compatibility with MongoDB tools while unlocking the power of SQL and Oracle’s AI-driven features. By following the steps outlined above, you can migrate your JSON data, create SQL views, and perform advanced analytics with ease. For detailed instructions, refer to the Oracle MongoDB API Quick Start Guide.
