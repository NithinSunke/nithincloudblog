---
title: "Install the AWS CLI on Windows"
author: "Sunke Nithin"
date: 2019-07-30
categories: ["AWS"]
tags: []
source: https://nithinsunke.wordpress.com/2019/07/30/install-the-aws-cli-on-windows/
---

# Install the AWS CLI on Windows

**Installing the AWS CLI**

1)download file from https://s3.amazonaws.com/aws-cli/AWSCLI64PY3.msi  
2)Run the downloaded MSI installer or the setup file.  
3)By default, the CLI installs to C:Program FilesAmazonAWSCLI (64-bit version) or C:Program Files (x86)AmazonAWSCLI (32-bit version).  
4)To confirm the installation, use the aws --version command at a command prompt (open the Start menu and search for cmd to start a command prompt).  

    C:> aws --version

    aws-cli/1.16.116 Python/3.6.8 Windows/10 botocore/1.12.106

  
**Uninstalling the AWS CLI**

To uninstall the AWS CLI, open the Control Panel, and then choose Programs and Features. Select the entry named AWS Command Line Interface, and then choose Uninstall to launch the uninstaller. Confirm that you want to uninstall the AWS CLI when you're prompted.

You can also launch the Programs and Features program from the command line with the following command.  

    C:> appwiz.cpl
