---
title: "How to Join Linux Servers into Windows Active Directory"
author: "Sunke Nithin"
date: 2025-04-08
categories: ["Linux"]
tags: ["active-directroy", "active-directroy", "ad", "DOCKER", "iam", "security", "technology", "ubuntu"]
source: https://nithinsunke.wordpress.com/2025/04/08/how-to-join-linux-servers-into-windows-active-directory/
---

# How to Join Linux Servers into Windows Active Directory

Integrating your Linux servers into a Windows Active Directory (AD) environment helps you unify identity management and enhance security. This post walks you through the steps to join a Linux server (RHEL-based) into a Windows AD domain.

## 🧠 Why Join Linux to AD?

- ✅ Centralized user management
- ✅ Single Sign-On (SSO)
- ✅ Group policy control
- ✅ Compliance and audit benefits

## 🧰 Prerequisites

- 🖥️ A working **Windows AD** domain (e.g., `example.local`)
- 🔑 Linux server (Ubuntu 20.04 / RHEL / Rocky / AlmaLinux)
- 🧑‍💼 Domain admin or equivalent privileges
- 🌐 Proper DNS settings pointing to the domain controller

### Step-by-Step: RHEL based systems

### install linux packages

``` wp-block-code
yum install sssd-ad sssd-tools realmd adcli oddjob oddjob-mkhomedir sssd samba-common-tools
```

### for oel8

``` wp-block-code
dnf install -y realmd sssd oddjob oddjob-mkhomedir adcli samba-common samba-common-tools krb5-workstation
```

join the linux system to AD domain

``` wp-block-code
realm  join --user=nsunke scslabs.com
```

To allow login using only nsunke (without @scslabs.com), follow these steps:

``` wp-block-code
sudo vi /etc/sssd/sssd.conf

use_fully_qualified_names = False

sudo systemctl restart sssd
```

test the connection

``` wp-block-code
ssh nsunke@linux-server.scslabs.com
```

## 🛡️ Bonus Tips

- Use `sssd` for caching credentials and offline login
- You can configure sudo access in `/etc/sudoers.d/`:

``` wp-block-code
%linuxadmin  ALL=(ALL) ALL
```

## 🧩 Conclusion

Integrating Linux servers into a Windows Active Directory environment bridges the gap between two powerful ecosystems. With tools like `realmd`, `SSSD`, and `adcli`, you can centralize authentication, simplify user management, and enhance security across your hybrid infrastructure.

Whether you're running Ubuntu, RHEL, or any other major Linux distribution, joining them to AD ensures consistent access policies and audit controls. This setup is especially valuable for enterprises looking to unify their IT landscape under a single identity management system.

## 📜 Disclaimer

The information provided in this blog post is for educational and informational purposes only. While every effort has been made to ensure accuracy, the configurations and commands shared here may vary based on your Linux distribution, domain environment, and system settings.

Please test any changes in a non-production or lab environment before applying them to live systems. The author and blog are not responsible for any issues, data loss, or misconfigurations that may arise from following this guide.

Always follow your organization’s IT policies and consult with your system administrator when in doubt.
