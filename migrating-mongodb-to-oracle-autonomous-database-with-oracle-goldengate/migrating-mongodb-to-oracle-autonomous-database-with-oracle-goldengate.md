---
title: "Migrating MongoDB to Oracle Autonomous Database with Oracle GoldenGate"
author: "Sunke Nithin"
date: 2025-09-04
categories: ["DATABASE"]
tags: ["MongoDB","ADB","Migration"]
---

# Migrating MongoDB to Oracle Autonomous Database with Oracle GoldenGate

Migrating a MongoDB database to Oracle Autonomous Database can unlock enterprise-grade performance, security, and AI capabilities while maintaining compatibility with existing MongoDB workloads. Oracle GoldenGate for MongoDB Migrations offers a streamlined, low-downtime solution to achieve this. In this blog post, we’ll walk through the step-by-step process of migrating MongoDB to Oracle Autonomous Database using Oracle GoldenGate, based on the detailed guidance from the Oracle GoldenGate YouTube video (https://www.youtube.com/watch?v=vMMwnjshu0A).

## Why Migrate to Oracle Autonomous Database?

Oracle Autonomous Database, particularly with Oracle Database 23ai, provides a cloud-native, serverless platform with:

- **MongoDB API Compatibility**: Run existing MongoDB applications with minimal or no code changes.
- **Performance and Scalability**: Leverage high-performance and serverless scaling.
- **Security**: Benefit from Oracle’s enterprise-grade security features.
- **AI Capabilities**: Utilize native AI features like Oracle Autonomous Database Select AI for advanced analytics.
- **Cost Efficiency**: Pay-per-use pricing reduces operational costs.

Oracle GoldenGate for MongoDB Migrations, available as a free tool for 183 days via the Oracle Container Registry, supports both offline and online migrations, making it ideal for mission-critical applications requiring minimal downtime.

## Prerequisites

Before starting, ensure you have:

1.  **MongoDB Source Database**: A MongoDB instance (version 3.6 or later) with data to migrate. For online migrations, configure it as a replica set with at least three replicas (this requirement will be reduced to one in future releases).
2.  **OCI Tenancy**: An Oracle Cloud Infrastructure (OCI) tenancy to create an Autonomous Database. Alternatively, use an existing Oracle Database 23ai instance outside OCI.
3.  **Docker Environment**: A Docker-compatible environment with x86 architecture (e.g., Podman on Linux).
4.  **Oracle Account**: Required to access the Oracle Container Registry.

## Step-by-Step Migration Process

### Step 1: Download and Set Up Oracle GoldenGate

- **Access Oracle Container Registry**:
  - Visit the Oracle Container Registry (https://container-registry.oracle.com/ords/ocr/ba/goldengate/goldengate-mongodb-migrations).
  - Sign in with an Oracle account or create one.
  - Accept the license agreement (covers 183 days of free usage).

![](images/image-434.png)

- **Generate a Secret Key Token**:
  - Create a token in the Oracle Container Registry to use as a password for Docker login.

![](images/image-436.png)

![](images/image-438.png)

copy the generated key and store in safe location

**Log in to Docker**:

- On your Docker host, run:

![](images/image-440.png)

Use your Oracle email as the username and the secret key token as the password.

**Run the GoldenGate Container**:

- Copy the docker run command from the registry page:

pull the docker image from repository

``` wp-block-code
docker pull
container-registry.oracle.com/goldengate/goldengate-mongodb-migrations:latest
```

![](images/image-442.png)

create a docker compose file docker-compose.yml

``` wp-block-code
services:
  goldengate-mongodb:
    container_name: goldengate-mongodb
    image: container-registry.oracle.com/goldengate/goldengate-mongodb-migrations:latest
    ports:
      - "9443:8443"
      - "443:443"
      - "80:80"
    environment:
      OGG_ADMIN: <admin_user_name>
      OGG_ADMIN_PWD: <admin_password>
    volumes:
      - u02_data:/u02
      - u03_data:/u03
      - ogg_certs:/etc/nginx/cert
    networks:
      - scslab
    restart: always

networks:
  scslab:
    external: true   # assumes you already created `scslab` network

volumes:
  u02_data:
  u03_data:
  ogg_certs:
```

run compose file

``` wp-block-code
docker compose up -d
```

![](images/image-450.png)

access the url

``` wp-block-code
http://ggmongodb01.scslabs.com:9443/
```

![](images/image-452.png)

Login with the credentials provided in the compose file.

![](images/image-454.png)

### Step 2: Create an Oracle Autonomous Database

**Navigate to OCI Console**:

- Log in to your OCI tenancy and go to **Oracle Database \> Autonomous Database**.

**Provision a Serverless Database**:

- Click **Create Autonomous Database**.
- Name the database and select **Oracle Database 23ai** as the version.
- Choose **JSON Database** as the workload type.
- Set an admin password and provide an email for notifications.
- Keep default settings for simplicity (creation takes ~40 seconds).

**Configure Network Access**:

- Set an Access Control List (ACL) to allow your Docker host’s IP address to access the database if required.
- Wait for the database to update.

**Enable MongoDB API**:

- In the OCI Console, go to the database’s **Tools \> MongoDB API** tab.
- Enable the MongoDB API (a simple toggle switch).
- Wait for the update to complete.
- Copy the MongoDB API connection URL (e.g., mongodb://\<host\>:\<port\>/?...).

![](images/image-456.png)

**Create a Migration Admin User**:

In the Autonomous Database, create a user (e.g., admin) with permissions to create MongoDB databases and collections:

``` wp-block-code
CREATE USER MONGOADM IDENTIFIED BY <password>;
GRANT CONNECT,RESOURCE,PDB_DBA TO MONGOADM;
```

This user will be used for the migration.

![](images/image-458.png)

Enable REST services

### Step 3: Verify Connections with MongoShell

Verify the connection by connecting to the target ATP database using mongodb API.

``` wp-block-code
mongosh 'mongodb://mongoadmin@<hostname>:<port>/?authSource=admin&authMechanism=SCRAM-SHA-256'
```

![](images/image-25.png)

I have connect to mongodb and autonomous database

### Step 4: Configure GoldenGate Connections

**Create Source Connection**:

- In GoldenGate Studio, go to **Connections \> Create Connection**.
- Name the connection (e.g., MongoDB_Source).
- Select **MongoDB** as the source type.
- Enter the MongoDB connection URL (e.g., mongodb://\<user\>:\<password\>@\<source_host\>:27017/?replicaSet=rs0), ensuring no database name is included before the ?.
- Provide the username and password for the source MongoDB.

![](images/image-49.png)

![](images/image-468.png)

**Create Target Connection**:

![](images/image-47.png)

![](images/image-39.png)

### Step 5: Create and Run the Migration Pipeline

**Create a Pipeline**:

- In GoldenGate Studio, go to **Home \> Create Pipeline**.
- Select the **Database Migration** recipe.
- Name the pipeline and choose **Online** migration to minimize downtime.

![](images/image-43.png)

![](images/image-484.png)

![](images/image-486.png)

![](images/image-488.png)

![](images/image-490.png)

![](images/image-492.png)

![](images/image-45.png)

![](images/image-51.png)

**Start the Migration**:

- Click **Start** to begin the migration.
- The initial load extracts data from the source and restores it to the target (~30 seconds for small databases).
- GoldenGate then starts the **Extract** and **Replicat** processes for ongoing replication.

![](images/image-53.png)

![](images/image-55.png)

![](images/image-57.png)

![](images/image-59.png)

![](images/image-61.png)

![](images/image-63.png)

inserted a record/document at source mongodb database

![](images/image-65.png)

you can view the new record count in gg studio which captured changes and applied at target.

![](images/image-67.png)

**Post-Migration and Switchover**

- Verify data in ATP (query JSON collections via SQL or MongoDB API).
- For online: Stop source apps, confirm sync (zero lag), stop pipeline, switch apps to ATP.
- Clean up: Delete pipeline (optionally delete GG config), stop container.

![](images/image-69.png)

![](images/image-71.png)

![](images/image-73.png)

### Conclusion

The migration of MongoDB to Oracle Autonomous Transaction Processing (ATP) using Oracle GoldenGate Studio offers a robust and streamlined solution for transferring data, whether through an offline or online approach. By leveraging the "Database Migration" recipe within GoldenGate Studio, users can efficiently perform an initial data load followed by real-time replication (for online migrations), minimizing downtime and ensuring data integrity. The process, detailed in the "Using Oracle GoldenGate for MongoDB Migrations" documentation (version 23ai, July 2025), leverages a Docker-based deployment, making it accessible with minimal setup, provided the prerequisites—such as a MongoDB replica set, ORDS/REST-enabled ATP, and proper security configurations—are met.

Successful execution requires careful preparation, including validating source and target connections, configuring pipelines with appropriate mappings, and monitoring the migration process to address potential issues like ORDS errors or UI unresponsiveness. The integration with the GoldenGate Microservices console enhances monitoring capabilities, offering detailed insights into replication performance. Post-migration, verifying data in ATP and executing a smooth switchover complete the transition.

While GoldenGate Studio is a free tool with community support (no Oracle Support), its limitations—such as support for only MongoDB and Oracle JSON targets, reliance on CA-signed certificates, and lack of advanced features like Active Data Guard—should be considered. For organizations requiring a reliable, cost-effective migration tool with real-time capabilities, this solution is highly effective, provided the target environment is correctly configured. For further assistance, the Oracle GoldenGate for MongoDB Migrations Community Forum remains a valuable resource.
