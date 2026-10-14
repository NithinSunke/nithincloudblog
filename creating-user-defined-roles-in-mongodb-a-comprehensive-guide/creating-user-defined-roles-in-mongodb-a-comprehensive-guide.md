---
title: "Creating User-Defined Roles in MongoDB: A Comprehensive Guide"
author: "Sunke Nithin"
date: 2026-10-14
categories: ["Database"]
tags: ["MongoDB"]
---

# Creating User-Defined Roles in MongoDB: A Comprehensive Guide

MongoDB’s flexibility as a NoSQL database makes it a popular choice for managing large datasets, but securing access to those datasets is critical. User-defined roles in MongoDB allow administrators to create custom permissions tailored to specific application or organizational needs, offering granular control beyond built-in roles like read or readWrite. This blog post explains how to create and manage user-defined roles in MongoDB,.

## Why Create User-Defined Roles?

User-defined roles are essential when built-in roles are too broad or restrictive for your use case. They enable precise control over database operations, ensuring users only perform authorized actions. Key scenarios include:

- **Granular Access Control**: Restrict users to specific actions (e.g., find, insert) on particular collections or databases.
- **Application-Specific Permissions**: Tailor roles for applications, such as allowing a logging service to write to a specific collection.
- **Compliance and Security**: Meet regulatory requirements by limiting access to sensitive data.
- **Team-Based Permissions**: Assign unique permissions to different teams, such as developers or analysts.

User-defined roles are ideal for scenarios requiring collection-level access control.

## Prerequisites

Before creating user-defined roles, ensure:

- MongoDB is running with authentication enabled (**--auth** flag).
- You have admin access to the admin database or a user with the **userAdminAnyDatabase** or **userAdmin** role.
- You understand the target database, collections, and actions for the custom role.

## Step-by-Step Guide to Creating User-Defined Roles

### Step 1: Connect to the MongoDB Shell

Connect to your MongoDB instance and authenticate as an admin user:

``` wp-block-code
mongosh -u mongoadm -p
```

Only users with appropriate privileges, such as userAdminAnyDatabase, root can create roles.

### Step 2: Switch to the Target Database

Roles are defined in the database where they will be applied. Switch to the desired database:

``` wp-block-code
use myDatabase
```

### Step 3: Create a User-Defined Role

Use the **db.createRole()** command to define a custom role. Specify the role name, privileges, and any inherited roles. The syntax is:

``` wp-block-code
db.createRole({
  role: "<roleName>",
  privileges: [
    {
      resource: { db: "<database>", collection: "<collection>" },
      actions: ["<action1>", "<action2>", ...]
    }
  ],
  roles: [
    { role: "<inheritedRole>", db: "<database>" }
  ]
})
```

**Example**: Create a role logWriter that allows reading from dataCollection and inserting into logCollection:

``` wp-block-code
db.createRole({
  role: "logWriter",
  privileges: [
    { resource: { db: "myDatabase", collection: "dataCollection" }, actions: ["find"] },
    { resource: { db: "myDatabase", collection: "logCollection" }, actions: ["insert"] }
  ],
  roles: []
})
```

You can also inherit existing roles to combine permissions:

``` wp-block-code
db.createRole({
  role: "logWriterWithRead",
  privileges: [
    { resource: { db: "myDatabase", collection: "logCollection" }, actions: ["insert"] }
  ],
  roles: [{ role: "read", db: "myDatabase" }]
})
```

This example inherits the read role for broader read access while adding specific insert privileges.

### Step 4: Assign the Role to a User

Assign the custom role to a new or existing user:

``` wp-block-code
// For a new user
db.createUser({
  user: "logUser",
  pwd: "xxxxxxx",
  roles: [{ role: "logWriter", db: "myDatabase" }]
})

// For an existing user
db.grantRolesToUser("existingUser", [{ role: "logWriter", db: "myDatabase" }])
```

Smarty Tech Fizz’s video provides a practical example of creating and assigning collection-specific roles.

### Step 5: Verify the Role

Check the role’s details to ensure it was created correctly:

``` wp-block-code
db.getRole("logWriter", { showPrivileges: true })
```

Then, verify allowed actions:

``` wp-block-code
use myDatabase
db.dataCollection.find() // Should succeed
db.logCollection.insertOne({ event: "Log entry" }) // Should succeed
db.dataCollection.insertOne({ name: "Test" }) // Should fail
```

### Step 6: Modify or Drop a Role

To update a role’s privileges, use db.updateRole():

``` wp-block-code
db.updateRole("logWriter", {
  privileges: [
    { resource: { db: "myDatabase", collection: "dataCollection" }, actions: ["find", "update"] },
    { resource: { db: "myDatabase", collection: "logCollection" }, actions: ["insert"] }
  ],
  roles: []
})
```

To delete a role, use db.dropRole():

``` wp-block-code
db.dropRole("logWriter")
```

Be cautious, as dropping a role affects all users assigned to it.

## Common Privilege Actions

When defining privileges, you can use actions like:

- find: Read data from a collection.
- insert: Add new documents.
- update: Modify existing documents.
- remove: Delete documents.
- createIndex: Create indexes.
- viewRole: View role definitions.

See the MongoDB Privilege Actions for a complete list.

## Best Practices

- **Minimal Privileges**: Grant only the necessary actions to reduce security risks.
- **Descriptive Role Names**: Use clear names (e.g., logWriter, dataReader) to reflect the role’s purpose.
- **Inherit Built-In Roles**: Combine custom privileges with built-in roles for efficiency.
- **Test Thoroughly**: Validate roles by testing with a user account to confirm expected behavior.
- **Secure Admin Access**: Restrict role creation to users with userAdmin or userAdminAnyDatabase roles.
- **Document Roles**: Maintain records of custom roles for team collaboration and auditing.

## Troubleshooting

- **Permission Denied**: Ensure the admin user has userAdmin or userAdminAnyDatabase privileges.
- **Role Not Found**: Verify the role was created in the correct database with db.getRole().
- **User Access Issues**: Check role privileges with db.getRole("\<role\>", { showPrivileges: true }).
- **Authentication Errors**: Confirm the user authenticates against the correct database using --authenticationDatabase.

## Additional Resources

- MongoDB Documentation: db.createRole()

## Conclusion

Creating user-defined roles in MongoDB with db.createRole() empowers administrators to enforce precise access control, enhancing security and compliance. By tailoring permissions to specific collections or actions, you can meet complex application needs while maintaining a secure environment.
