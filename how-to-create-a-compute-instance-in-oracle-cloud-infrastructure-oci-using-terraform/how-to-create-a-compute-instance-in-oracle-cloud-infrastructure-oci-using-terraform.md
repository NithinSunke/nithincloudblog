---
title: "How to Create a Compute Instance in Oracle Cloud Infrastructure (OCI) using Terraform"
author: "Sunke Nithin"
date: 2025-04-14
categories: ["Terraform"]
tags: ["CLOUD", "devops", "iac", "oci"]
source: https://nithinsunke.wordpress.com/2025/04/14/how-to-create-a-compute-instance-in-oracle-cloud-infrastructure-oci-using-terraform/
---

# How to Create a Compute Instance in Oracle Cloud Infrastructure (OCI) using Terraform

In this post, we’ll walk through the process of provisioning a Compute Instance in OCI using Terraform, covering everything from setting up the provider to defining instance configurations and deploying resources.

### 📖 Table of Contents

1.  [Prerequisites](#prerequisites)
2.  [Terraform Provider Setup](#terraform-provider-setup)
3.  [Fetch Latest Oracle Linux Image](#fetch-latest-oracle-linux-image)
4.  [Define Compute Instance Resource](#define-compute-instance-resource)
5.  [Apply Terraform Configuration](#apply-terraform-configuration)
6.  [Clean Up Resources](#clean-up-resources)
7.  [Conclusion](#conclusion)

## 📌 Prerequisites

Before we begin, ensure you have the following:

- **Oracle Cloud Account**
- **OCI CLI installed and configured**
- **Terraform installed** (v1.6+ recommended)
- **API Signing Keys configured**
- A **compartment OCID** and **subnet OCID**

## Terraform Provider Setup

Create a file named `provider.tf`:

``` wp-block-code
terraform {
  required_providers {
    oci = {
      source = "oracle/oci"
    }
  }
}

provider "oci" {
  region              = var.region
  config_file_profile = var.config_file_profile
}
```

create variables.tf to hold variables

``` wp-block-code
#variable for region
variable "region" {
  description = "OCI region to deploy the compute instance"
  type        = string
  default     = "us-phoenix-1"
}

#variable for config_file_profile
variable "config_file_profile" {
  description = "OCI config file profile to use for authentication"
  type        = string
  default     = "DEFAULT"
}

#variable for availability_domain
variable "availability_domain" {
  description = "Availability domain for the compute instance"
  type        = string
  default     = "kWcK:PHX-AD-1"
}

#variable for compartment_id
variable "compartment_id" {
  description = "Compartment ID where the compute instance will be created"
  type        = string
  default     = "ocid1.compartment.oc1.."
}

#variable for shape
variable "shape" {
  description = "Shape of the compute instance"
  type        = string
  default     = "VM.Standard3.Flex"
}

#variable for cpus
variable "ocpus" {
  description = "Number of OCPUs for the compute instance"
  type        = number
  default     = 2
}

#variable for memory_in_gbs
variable "memory_in_gbs" {
  description = "Memory in GBs for the compute instance"
  type        = number
  default     = 8
}

#variable for instance_name 
variable "instance_name" {
  description = "Name of the compute instance"
  type        = string
  default     = "phoenix_instance"
}

#variable for vnic_hostname_label
variable "vnic_hostname_label" {
  description = "Hostname label for the VNIC"
  type        = string
  default     = "phoenix-instance"
}    

#variable for vnic_display_name
variable "vnic_display_name" {
  description = "Display name for the VNIC"
  type        = string
  default     = "phoenix_instance_vnic"
}

#variable for ssh_public_key_path
variable "ssh_public_key_path" {
  description = "Path to the SSH public key file"
  type        = string
  default     = "C:\\Users\\Nithin Sunke\\.ssh\\ssh-key-2024-08-27.key.pub"
}

#variable for subnet_id
variable "subnet_id" {
  description = "Subnet ID for the compute instance"
  type        = string
  default     = "ocid1.subnet.oc1.phx."
}
```

main.tf file

``` wp-block-code
data "oci_core_images" "latest_ol8_image" {
  compartment_id = var.compartment_id
  operating_system  = "Oracle Linux"
  operating_system_version = "8"
  shape = "VM.Standard3.Flex"
  sort_by  = "TIMECREATED"
  sort_order = "DESC"
}
output "latest_ol8_image_ocid" {
  value = data.oci_core_images.latest_ol8_image.images[0].id
}

resource "oci_core_instance" "phoenix_instance" {
  availability_domain = var.availability_domain
  compartment_id     = var.compartment_id
  display_name       = var.instance_name
  shape             = var.shape
  shape_config {
    ocpus         = var.ocpus
    memory_in_gbs = var.memory_in_gbs
  }
  create_vnic_details {
    subnet_id              = var.subnet_id
    assign_public_ip       = false
    display_name           = var.vnic_display_name
    hostname_label         = var.vnic_hostname_label
  }
  source_details {
    source_type = "image"
    source_id   = data.oci_core_images.latest_ol8_image.images[0].id
    boot_volume_size_in_gbs = 50
  }
  
  metadata = {
    ssh_authorized_keys = file(var.ssh_public_key_path)
  }
}
```

## 📌 Apply Terraform Configuration

Run the following commands:

``` wp-block-code
terraform init
terraform plan
terraform apply
```

Type `yes` when prompted to approve the changes.

## Clean Up Resources

When you’re done, you can destroy the created resources:

``` wp-block-code
terraform destroy
```

## Conclusion

This method allows for repeatable, infrastructure-as-code deployment and integrates neatly into your cloud automation workflows.

## 🔗 Useful References

- [OCI Terraform Provider Docs](https://registry.terraform.io/providers/oracle/oci/latest)
- [OCI CLI Configuration](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm)
- [Oracle Cloud Pricing](https://www.oracle.com/cloud/price-list/)
