---
title: "Connecting to MongoDB on Oracle Enterprise Linux 8: Shell and GUI Methods"
author: "Sunke Nithin"
date: 2025-08-25
categories: ["MongoDB"]
tags: ["devops", "DOCKER", "Linux", "technology", "ubuntu"]
source: https://nithinsunke.wordpress.com/2025/08/25/connecting-to-mongodb-on-oracle-enterprise-linux-8-shell-and-gui-methods/
---

# Connecting to MongoDB on Oracle Enterprise Linux 8: Shell and GUI Methods

Following our previous blog post on installing MongoDB on Oracle Enterprise Linux 8 (OEL 8) and creating an admin user (How to Install MongoDB on OEL 8), this guide explains how to connect to your MongoDB database using both the command-line shell (mongosh) and a graphical user interface (GUI) tool, MongoDB Compass. We’ll use the admin user (mongoadmin) created in the previous post to demonstrate secure connections. This tutorial assumes MongoDB 8.0 is installed on OEL 8 with the Red Hat Compatible Kernel (RHCK) and authentication enabled.

## Prerequisites

- MongoDB 8.0 Community Edition installed on OEL 8 (see previous blog post).
- Admin user (mongoadmin) created with a secure password.
- MongoDB service running (sudo systemctl status mongod).
- Internet connection for downloading MongoDB Compass (for GUI method).
- For remote connections: MongoDB configured with bindIp: 0.0.0.0 in /etc/mongod.conf and port 27017 open in the firewall.
- Administrative privileges (root or sudo) for installing tools.

## Method 1: Connecting Using the MongoDB Shell (mongosh)

The mongosh shell is a powerful command-line interface for interacting with MongoDB. Follow these steps to connect using the admin user.

### Step 1: Verify MongoDB Service

Ensure MongoDB is running:

``` wp-block-syntaxhighlighter-code
sudo systemctl status mongod
```

If inactive, start it:

``` wp-block-syntaxhighlighter-code
sudo systemctl start mongod
```

### Step 2: Connect to MongoDB

Use mongosh to connect with the mongoadmin user:

``` wp-block-syntaxhighlighter-code
mongosh -u mongoadmin -p --authenticationDatabase admin
```

- -u mongoadmin: Specifies the admin username.
- -p: Prompts for the password (enter the password set during user creation).
- --authenticationDatabase admin: Specifies the admin database where the user is defined.

### Step 3: Verify the Connection

Run these commands to confirm the connection:

``` wp-block-syntaxhighlighter-code
db.version()
```

This displays the MongoDB version (e.g., 8.0.13).

``` wp-block-syntaxhighlighter-code
db
```

This shows the current database (admin).

### Step 4: Perform a Test Operation

To test, switch to a new database and insert a document:

``` wp-block-syntaxhighlighter-code
use myDatabase
db.myCollection.insertOne({ name: "Test User", role: "Admin" })
```

The mongoadmin user has readWriteAnyDatabase privileges, allowing operations on any database. To view the inserted document:

``` wp-block-syntaxhighlighter-code
db.myCollection.find().pretty()
```

### Step 5: Exit the Shell

When done, exit the shell:

``` wp-block-syntaxhighlighter-code
exit
```

### Remote Connection (Optional)

If MongoDB is configured for remote access (check /etc/mongod.conf for bindIp: 0.0.0.0 and ensure port 27017 is open), connect from another machine:

``` wp-block-syntaxhighlighter-code
mongosh -u mongoadmin -p --host <OEL8_SERVER_IP> --port 27017 --authenticationDatabase admin
```

Replace \<OEL8_SERVER_IP\> with your server’s IP address.

## Method 2: Connecting Using MongoDB Compass (GUI)

MongoDB Compass is a graphical tool for managing MongoDB databases, ideal for users who prefer a visual interface. Here’s how to set it up and connect.

### Step 1: Download and Install MongoDB Compass

- **Download MongoDB Compass**:
  - Visit the MongoDB Compass download page.
  - Select the version for your operating system (e.g., Linux for a local OEL 8 machine or Windows/Mac for remote management).
  - For OEL 8, download the .rpm package (e.g., mongodb-compass-1.44.6-x86_64.rpm as of August 2025).
- **Install on OEL 8**:
  - Copy the downloaded .rpm file to your OEL 8 server (e.g., via scp or a file transfer tool). Then, install it:

``` wp-block-code
sudo yum install -y ./mongodb-compass-1.44.6-x86_64.rpm
```

Accept any GPG key prompts.

- **Launch MongoDB Compass**:
  - If running on OEL 8 with a desktop environment, start Compass:

``` wp-block-code
mongodb-compass
```

If managing remotely, install Compass on your local machine (Windows, Mac, or Linux) and proceed to the next step.

Step 2: Connect to MongoDB

- Open MongoDB Compass.
- In the **New Connection** window, enter the connection string:

![](../images/2025/08/image-351.png)

![](../images/2025/08/image-354.png)

![](../images/2025/08/image-355.png)

- Click **Connect**. If successful, Compass displays the MongoDB instance with databases like admin, config, and local.

### Step 3: Explore and Manage Databases

- Use the Compass interface to view databases, collections, and documents.
- Test by creating a new database (e.g., myDatabase) and collection (e.g., myCollection) via the GUI, then insert a document:

``` wp-block-code
{ "name": "GUI Test", "role": "User" }
```

![](../images/2025/08/image-357.png)

![](../images/2025/08/image-359.png)

![](../images/2025/08/image-361.png)

## Troubleshooting

- **Shell Connection Issues**:
  - **Authentication Failed**: Verify the username (mongoadmin), password, and --authenticationDatabase admin. If the password is forgotten, disable authentication temporarily in /etc/mongod.conf (remove security.authorization), restart MongoDB, reset the password, and re-enable authentication.
  - **Connection Refused**: Ensure MongoDB is running (sudo systemctl status mongod) and port 27017 is open for remote connections:
  - **`sudo firewall-cmd --list-ports`**
  - If needed, open the port:
  - **`sudo firewall-cmd --add-port=27017/tcp --permanent sudo firewall-cmd --reload`**
  - Check logs for errors:`sudo cat /var/log/mongodb/mongod.log`
- **Compass Connection Issues**:
  - Ensure the connection string includes ?authSource=admin.
  - For remote connections, verify bindIp: 0.0.0.0 in /etc/mongod.conf and that port 27017 is accessible.
  - If Compass fails to launch on OEL 8, ensure dependencies are met (sudo yum install -y libXScrnSaver for common GUI issues).

## Security Notes

- Always use strong passwords for the mongoadmin user.
- For production, enable TLS/SSL for secure connections (refer to MongoDB Documentation).
- Limit remote access by configuring firewall rules to allow only specific IP addresses.

## Conclusion

You’ve now learned how to connect to MongoDB on OEL 8 using both the mongosh shell and MongoDB Compass GUI. The shell is ideal for scripting and quick operations, while Compass offers a user-friendly interface for visual database management. For advanced tasks like replica sets or sharding, consult the MongoDB Documentation.
