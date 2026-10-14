---
title: "Vagrant Installation & Configuration"
author: "Sunke Nithin"
date: 2024-05-20
categories: ["Uncategorized"]
tags: []
source: https://nithinsunke.wordpress.com/2024/05/20/vagrant-installation-configuration/
---

# Vagrant Installation & Configuration

Step 1 – Download Vagrant URL – https://www.vagrantup.com/downloads Step 2 –  
Install Vagarnt in Windows

[![](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-1.jpg)](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-1.jpg)

[![](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-2.jpg)](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-2.jpg)

[![](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-3.jpg)](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-3.jpg)

[![](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-4.jpg)](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-4.jpg)

[![](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-5.jpg)](https://www.devopsschool.com/blog/wp-content/uploads/2022/04/vagrant-install-windows-5.jpg)

Step 3 – Verify Vagrant  
\$ vagrant --version  
\$ vagrant --help \$

Step 4 – Basic Workflow in Vagrant  
\$ mkdir vagrant-vms  
\$ cd vagrant-vms/  
\$ vagrant box add centos-min https://github.com/tommy-muehle/puppet-vagrant-boxes/releases/download/1.1.0/centos-7.0-x86_64.box  
\$ vagrant --help  
\$ vagrant box --help  
\$ vagrant box list  
\$ vagrant init centos-min  
\$ vagrant up  
\$ vagrant ssh  
\$ vagrant halt  
\$ vagrant destroy
