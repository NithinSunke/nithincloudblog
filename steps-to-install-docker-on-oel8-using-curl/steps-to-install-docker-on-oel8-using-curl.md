---
title: "Steps to Install Docker on OEL8 Using Curl"
author: "Sunke Nithin"
date: 2025-08-08
categories: ["DOCKER"]
tags: []
source: https://nithinsunke.wordpress.com/2025/08/08/steps-to-install-docker-on-oel8-using-curl/
---

# Steps to Install Docker on OEL8 Using Curl

- . **Prepare the system**

``` wp-block-code
sudo dnf install -y dnf-utils device-mapper-persistent-data lvm2
```

These are required by Docker for managing storage.

- . **Add Docker’s official repository**

You can use curl + dnf to add the Docker repo.

``` wp-block-code
sudo curl -o /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
```

✅Even though it's CentOS repo, it works perfectly on OEL8 because OEL8 is compatible

- **Disable the CentOS Extras repo inside the Docker repo (optional but clean)**

``` wp-block-code
sudo sed -i 's/enabled=1/enabled=0/' /etc/yum.repos.d/docker-ce.repo
```

We will explicitly enable the stable repo when installing to avoid conflicts.

- **Install Docker Engine**

``` wp-block-code
sudo dnf --enablerepo=docker-ce-stable install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**This installs:**

- **Docker engine**
- **Docker CLI**
- **Containerd (required)**
- **Buildx (for building images)**
- **Docker Compose (plugin style)**

**Start and Enable Docker service**

``` wp-block-code
sudo systemctl start docker
sudo systemctl enable docker
```

**Verify Docker is Running**

``` wp-block-code
sudo docker version
sudo docker info
```

**(Optional) Run Docker without sudo**

**To allow your user to run docker commands without sudo:**

``` wp-block-code
sudo usermod -aG docker $USER
newgrp docker
```

(You need to logout and login again if newgrp doesn't apply immediately.)
