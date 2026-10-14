---
title: "Enable Authentication in MongoDB"
author: "Sunke Nithin"
date: 2025-08-26
categories: ["Database"]
tags: ["MongoDB"]
---

# Enable Authentication in MongoDB

By default, MongoDB installs with authentication **disabled**, which means anyone can connect and access your database without credentials. For security reasons, especially in production environments, it is highly recommended to **enable authentication**.

In this blog, I’ll walk you through the steps to enable MongoDB authentication and secure your database with a username and password.

**Start MongoDB without Authentication**

When you first install MongoDB, authentication is turned off. This allows you to create the initial admin user.

``` wp-block-code
systemctl start mongod
```

**Connect to MongoDB**

``` wp-block-code
mongosh
```

Switch to the `admin` Database

``` wp-block-code
use admin
```

**Create an Admin User**

Create a root user with a strong username and password:

``` wp-block-code
db.createUser({
  user: "mongoadm",
  pwd: "xxxxxxxxx",
  roles: [ { role: "root", db: "admin" } ]
})
```

This creates a root user who can manage all databases.

Enable Authentication in MongoDB Configuration

Edit your MongoDB configuration file (usually **`/etc/mongod.conf`**):

``` wp-block-code
security:
  authorization: enabled  --add this line to enable authorization
```

Save and exit.

**Restart MongoDB**

Restart the MongoDB service to apply the changes:

``` wp-block-code
sudo systemctl restart mongod
```

**Connect with Authentication**

Now that authentication is enabled, you must log in with a user account:

different ways to connect to mongodb

``` wp-block-code
# full command specifying password in the shell itself (not Safe)
mongosh -u "mongoadmin" -p "xxxxx" --authenticationDatabase "admin"

# authentication against admin database it will ask for user password to connect
mongosh -u "mongoadmin" -p  --authenticationDatabase "admin"

# by default if we not specify authenticationDatabase it will look in admin database if user is created under other databases it will fail
 mongosh -u "mongoadmin" -p 
```

## Best Practices

- Always use **strong passwords** for database users.
- Create **application-specific users** with only the required roles (avoid using `root` everywhere).
- Never expose MongoDB directly to the internet without proper firewalls or VPN.
- Regularly rotate passwords and monitor login attempts.

## Conclusion

Enabling authentication is the **first step to securing your MongoDB environment**. Without it, anyone could connect and access your data. By following these steps, you ensure that only authorized users can interact with your database.
