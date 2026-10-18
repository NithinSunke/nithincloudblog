---
title: "How to Change Docker Data Root (Storage Location)"
author: "Sunke Nithin"
date: 2025-08-08
categories: ["DOCKER"]
tags: []
source: https://nithinsunke.wordpress.com/2025/08/08/how-to-change-docker-data-root-storage-location/
---

# How to Change Docker Data Root (Storage Location)

Suppose you want to move Docker storage to /mnt/docker_storage.

## Steps:

1.  **Create New Directory**

``` wp-block-code
sudo mkdir -p /mnt/docker_storage
```

Make sure /mnt is a separate disk or mount point with enough space.

2\. **Stop Docker Service**

``` wp-block-code
sudo systemctl stop docker
```

3\. **Copy Existing Docker Data (optional)**

If Docker is already installed and you have some images/containers, copy existing data:

``` wp-block-code
sudo rsync -aP /var/lib/docker/ /mnt/docker_storage/
```

-aP preserves permissions and shows progress.

4\. **Configure Docker to Use New Directory**

Edit or create the daemon.json config file:

``` wp-block-code
{
 "data-root": "/mnt/docker_storage"
}
```

This tells Docker to use /mnt/docker_storage instead of /var/lib/docker

5\. **Start Docker Service**

``` wp-block-code
sudo systemctl daemon-reexec
sudo systemctl start docker
```

**6. Verify New Storage Location**

``` wp-block-code
docker info | grep "Docker Root Dir"
```

output:

``` wp-block-code
Docker Root Dir: /mnt/docker_storage
```
