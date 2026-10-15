---
title: "Steps to Upgrade vCenter Server 7 to vCenter 8"
author: "Sunke Nithin"
date: 2025-05-04
categories: ["Virtualization"]
tags: ["CLOUD", "nsx", "vcf", "vmware", "vsphere"]
source: https://nithinsunke.wordpress.com/2025/05/04/steps-to-upgrade-vcenter-server-7-to-vcenter-8/
---

# Steps to Upgrade vCenter Server 7 to vCenter 8

With the release of **vCenter Server 8**, VMware has introduced several performance improvements, security enhancements, and support for modern infrastructure. If you're running **vCenter 7**, it's a great time to upgrade and take advantage of these new features.

In this post, we'll walk through the **vCenter 7 to vCenter 8 upgrade** process using the vCenter Server Appliance (VCSA) UI installer.

## Pre-Upgrade Checklist

Before upgrading, make sure you’ve done the following:

- Take a **full backup** or snapshot of the existing VCSA.
- Verify **vCenter 8 compatibility** with your ESXi hosts and hardware via the VMware Compatibility Guide.
- Ensure you're running at least **vCenter 7.0 Update 3** (minimum required).
- Download the **vCenter Server 8 ISO** from VMware Customer Connect.
- Create a temporary static IP and DNS entry for the new vCenter appliance (used during upgrade).

## Step 1: Mount vCenter 8 ISO and Start Installer

Mount the **VC 8 ISO** on your workstation.

Navigate to: bashCopyEdit

``` wp-block-code
/vcsa-ui-installer/win32
```

Launch `installer.exe` and choose **Upgrade**.

![](images/image-74.png)

![](images/image-75.png)

## Step 2: Stage 1 – Deploy New vCenter Appliance

Review the introduction and click **Next**.

![](images/image-76.png)

![](images/image-77.png)

Accept the EULA.

![](images/image-78.png)

Source vCenter Server credentials (vCenter 7)

![](images/image-79.png)

ESXi host credentials where the source VCSA runs

Target ESXi host credentials where the **new VCSA 8** will be deployed

![](images/image-80.png)

Confirm certificates and continue.

![](images/image-81.png)

Enter the **new VCSA name**, root password, and deployment size (Tiny/Small/etc.).

![](images/image-82.png)

![](images/image-83.png)

![](images/image-84.png)

![](images/image-85.png)

Choose the **datastore** for deployment.

1.  Configure network for **temporary VCSA 8**:
    - Temporary hostname
    - Static IP address
    - Subnet, gateway, DNS

![](images/image-86.png)

📦 Click **Finish** to deploy the new appliance.

![](images/image-87.png)

![](images/image-88.png)

![](images/image-89.png)

![](images/image-90.png)

## Step 3: Stage 2 – Transfer Configuration

After deployment, the wizard moves to **Stage 2**.

![](images/image-91.png)

![](images/image-92.png)

![](images/image-93.png)

![](images/image-94.png)

Select the **data to transfer** (configuration only or config + historical data).

1.  Choose whether to enable **CEIP and VMware analytics**.
2.  Click **Finish** to begin the data migration and cutover.

![](images/image-95.png)

![](images/image-96.png)

![](images/image-97.png)

![](images/image-98.png)

![](images/image-99.png)

![](images/image-100.png)

![](images/image-101.png)

⏳ This process will shut down the old vCenter 7 appliance and power on the new vCenter 8.

## Post-Upgrade Steps

- Log in to `https://<new-vcenter-FQDN>:443` using `administrator@vsphere.local`
- Verify:
  - Inventory (clusters, hosts, VMs)
  - Licenses
  - vDS and storage configurations
- Reconfigure backup tools and monitoring agents
- Delete the old VCSA if everything is verified

![](images/image-102.png)

### Tips & Notes

The old appliance is **not deleted** automatically—cleanup is manual.

During the upgrade, **do not power off** any VMs or hosts.

Keep both appliances on the same subnet for smoother migration.

## Conclusion

Upgrading from **vCenter 7 to 8** is a safe and structured process with VMware’s two-stage installer. With just a few clicks, you can benefit from the latest vSphere innovations. Always test in a lab before upgrading production environments!
