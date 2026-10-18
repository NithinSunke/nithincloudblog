---
title: "How to Configure Cloud-Init in Proxmox VE"
author: "Sunke Nithin"
date: 2025-04-22
categories: ["Virtualization"]
tags: []
source: https://nithinsunke.wordpress.com/2025/04/22/how-to-configure-cloud-init-in-proxmox-ve/
---

# How to Configure Cloud-Init in Proxmox VE

** Step 1: Install Cloud-Init in the VM OS**

- Install your preferred Linux distribution (like Ubuntu, CentOS, or Oracle Linux).
- Inside the VM, install Cloud-Init:

**For Ubuntu/Debian:**

``` wp-block-code
sudo apt update
sudo apt install cloud-init
```

**For CentOS/RHEL/OEL:**

``` wp-block-code
sudo dnf install cloud-init
sudo systemctl enable cloud-init
```

- Shutdown the VM.

**Step 2: Convert VM to a Template**

In the Proxmox GUI:

1.  Select the VM.
2.  Go to **More → Convert to Template**.

Now your VM is a template that can be cloned.

**Step 3: Add a Cloud-Init Drive**

1.  Select your VM template.
2.  Go to **Hardware → Add → CloudInit Drive**.
3.  Select storage (preferably local-lvm or local).
4.  Click **Add**.

**Step 4: Configure Cloud-Init Settings**

1.  Go to the **Cloud-Init** tab in your VM.
2.  Set:
    - Hostname
    - User
    - Password
    - SSH public key (if needed)
    - IP configuration (DHCP or Static)

**Step 5: Clone a New VM from Template**

1.  Right-click the template → **Clone**
2.  Choose:
    - VM ID
    - Name
    - Mode: **Full Clone** or **Linked Clone**
3.  Click **Clone**

**Step 6: Start the New VM**

- On boot, Cloud-Init reads the configuration and applies:
  - Hostname
  - Networking
  - SSH keys
  - Password

✅ You have a fully configured VM ready in seconds.

**📑** **Bonus: View Cloud-Init Logs inside the VM**

After first boot:

``` wp-block-code
sudo cloud-init status
sudo cloud-init logs
```

**🎁** **Optional: Automating Network via Cloud-Init**

You can preconfigure static IP or DHCP via the Cloud-Init tab in Proxmox, or using a network-config file in /etc/cloud/cloud.cfg.d/.

**📌** **Summary:**

|                               |                             |
|-------------------------------|-----------------------------|
| **Task**                      | **Where to Do It**          |
| Install Cloud-Init            | Inside VM before templating |
| Add Cloud-Init drive          | In Proxmox VM Hardware tab  |
| Configure Cloud-Init settings | In Proxmox Cloud-Init tab   |
| Clone and start VM            | In Proxmox GUI              |
