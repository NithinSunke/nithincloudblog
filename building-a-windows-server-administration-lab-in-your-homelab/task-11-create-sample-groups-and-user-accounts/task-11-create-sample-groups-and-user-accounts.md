---
title: "Task 11- Create sample groups and user accounts"
author: "Sunke Nithin"
date: 2025-05-08
categories: ["WINDOWS"]
tags: ["active-directory", "cybersecurity", "microsoft", "security", "technology"]
source: https://nithinsunke.wordpress.com/2025/05/08/task-11-create-sample-groups-and-user-accounts/
---

# Task 11- Create sample groups and user accounts

Creating sample user accounts is an essential part of building out your Active Directory domain. Whether you're setting up a homelab or preparing for enterprise deployment, organized test users help with validating login behavior, Group Policy Objects (GPOs), folder redirection, and access control.

In this guide, we’ll walk through how to create user accounts for multiple departments using PowerShell in a domain named **scslab.corp**.

**Goal**

We’ll create **5 user accounts** for each department:

- IT
- HR
- Sales

And **groups**

IT-Group,

HR-Group,

Sales-Group

**Script:**

Power shell script to create groups

``` wp-block-code
New-ADGroup -Name "IT-Group" -GroupScope Global -Path "OU=IT,OU=Departments,DC=scslab,DC=corp"

New-ADGroup -Name "HR-Group" -GroupScope Global -Path "OU=HR,OU=Departments,DC=scslab,DC=corp"

New-ADGroup -Name "Sales-Group" -GroupScope Global -Path "OU=Sales,OU=Departments,DC=scslab,DC=corp"
```

If bulk users are need to be onboarded create a csv file and give file as input to the powershell script.

\<\<all_department_users.csv\>\>

``` wp-block-code
# Import the CSV file containing user details

$csvPath = "C:\Users\Administrator\Documents\all_department_users.csv"  # Replace with the actual path to your CSV file

$users = Import-Csv -Path $csvPath

# Loop through each user in the CSV and create them

foreach ($user in $users) {

    $FullName = $user.'Full Name'

    $Username = $user.Name + "." + $user.Surname

    $Email = $user.Email

    $Department = $user.Department

    $GroupName = $user.Group

    $Password = ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force

    $OU = "OU=$Department,OU=Departments,DC=scslab,DC=corp"

    New-ADUser `

        -Name $FullName `

        -GivenName $user.Name `

        -Surname $user.Surname `

        -SamAccountName $Username `

        -UserPrincipalName $Email `

        -EmailAddress $Email `

        -Path $OU `

        -AccountPassword $Password `

        -Enabled $true `

        -PasswordNeverExpires $true `

        -ChangePasswordAtLogon $false

    Add-ADGroupMember -Identity $GroupName -Members $Username

}
```

**What This Script Does**

- Creates a user and assign it to the group.
- Places them in their respective departmental OU
- Assigns them a temporary password
- Requires password change at first login

**Verifying the Users**

Use Active Directory Users and Computers (ADUC) to browse each OU:

- Go to Start \> Administrative Tools \> Active Directory Users and Computers
- Expand scslab.corp \> Departments \> IT (or other departments)
- You’ll see the new users listed

**Summary**

Creating sample users in AD DS is easy and highly scriptable with PowerShell. Whether you're building a lab or setting up a production OU structure, this process gives you realistic test subjects to experiment with policies, access rights, and login procedures.
