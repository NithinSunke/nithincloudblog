---
title: "Oracle GoldenGate 23ai: Building a Docker Image from Oracle GitHub"
author: "Sunke Nithin"
date: 2025-04-28
categories: ["Goldengate", "Virtualization"]
tags: ["devops", "DOCKER", "kubernetes", "Linux", "technology"]
source: https://nithinsunke.wordpress.com/2025/04/28/oracle-goldengate-23ai-building-a-docker-image-from-oracle-github/
---

# Oracle GoldenGate 23ai: Building a Docker Image from Oracle GitHub

Oracle GoldenGate 23ai is the latest innovation in real-time data replication and streaming. If you're setting up a modernized, containerized GoldenGate environment, you’ll be happy to know that Oracle provides **official Docker build scripts** via their **GitHub repository**.

In this blog, I'll walk you through the **step-by-step process to build your own Oracle GoldenGate 23ai Docker image** from Oracle's GitHub project.

### **📥** **Step 1: Clone the Oracle GoldenGate GitHub Repository**

First, clone the Oracle GoldenGate container repository which contains Dockerfiles and setup scripts:

``` wp-block-code
git clone https://github.com/oracle/docker-images.git
```

Navigate to the GoldenGate directory:

``` wp-block-code
cd docker-images/OracleGoldenGate
```

Depending on your goldengate version, select the correct subfolder.

``` wp-block-code
cd 23
```

### **📦** **Step 2: Download Oracle GoldenGate 23ai Software**

Oracle cannot distribute the software binaries directly.

✅ So, **you must manually download** the GoldenGate 23ai for Oracle from [Oracle Software Delivery Cloud](https://edelivery.oracle.com/) or Oracle’s official site.

Once downloaded (example: fbo_ggs_Linux_x64_oracle_shiphome.zip), place it in the **Docker build directory**.

### **⚙️ Step 3: Build the Docker Image**

Now, you can build the Docker image using the below command:

``` wp-block-code
[root@docker 23]# ls
bin  Dockerfile  install-deployment.sh  install-prerequisites.sh  nginx  oracle_gg_23ai.zip  README.md  samples
[root@docker 23]# docker build --tag=oracle/goldengate:23ai --build-arg INSTALLER=oracle_gg_23ai.zip .
[+] Building 296.1s (12/12) FINISHED                                                                                                                docker:default
 => [internal] load build definition from Dockerfile                                                                                                          0.1s
 => => transferring dockerfile: 1.23kB                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/oraclelinux:8                                                                                              2.8s
 => [internal] load .dockerignore                                                                                                                             0.0s
 => => transferring context: 2B                                                                                                                               0.0s
 => [internal] load build context                                                                                                                            11.5s
 => => transferring context: 517.69MB                                                                                                                        11.5s
 => [1/7] FROM docker.io/library/oraclelinux:8@sha256:0d48f4c6bad271da7abae47242e0478805343b81ecd1cec59523a6012d47eb72                                       24.5s
 => => resolve docker.io/library/oraclelinux:8@sha256:0d48f4c6bad271da7abae47242e0478805343b81ecd1cec59523a6012d47eb72                                        0.0s
 => => sha256:32987477004d025c90425ee548b45c41681f7dcd24614487d8d5a4b01243e34f 580B / 580B                                                                    0.0s
 => => sha256:498c112f0dac2b610e642521f4ab03fd5ba6d5e7a1d29541ccde7d79d7d7dd9f 100.82MB / 100.82MB                                                           14.7s
 => => sha256:0d48f4c6bad271da7abae47242e0478805343b81ecd1cec59523a6012d47eb72 2.37kB / 2.37kB                                                                0.0s
 => => sha256:0c487426a266dea73c4c8f1fb717d44b41634449a214c82a1a8cf66f17bea40d 1.02kB / 1.02kB                                                                0.0s
 => => extracting sha256:498c112f0dac2b610e642521f4ab03fd5ba6d5e7a1d29541ccde7d79d7d7dd9f                                                                     9.5s
 => [2/7] RUN           : oracle_gg_23ai.zip                                                                                                                  1.4s
 => [3/7] COPY          install-*.sh      /tmp/                                                                                                               0.1s
 => [4/7] COPY          oracle_gg_23ai.zip      /tmp/installer.zip                                                                                            3.0s
 => [5/7] COPY          bin/              /usr/local/bin/                                                                                                     0.1s
 => [6/7] RUN           bash -c  /tmp/install-prerequisites.sh &&               bash -c  /tmp/install-deployment.sh    &&               rm   -fr /tmp/*     217.3s
 => [7/7] COPY          nginx/            /etc/nginx/                                                                                                         0.1s
 => exporting to image                                                                                                                                       46.5s
 => => exporting layers                                                                                                                                      46.5s
 => => writing image sha256:481060262c50ca37766f10baeef1273cc0b51dc98ff5cc56143eed8bf17db6ae                                                                  0.0s
 => => naming to docker.io/oracle/goldengate:23ai                                                                                                             0.0s
```

If successful, Docker will create an image like:

``` wp-block-code
oracle/goldengate:23ai
```

### Step 4: Run the Oracle GoldenGate 23ai Container using docker-compose file

Create a docker composefile

``` wp-block-code
[root@docker docker23c]# cat docker-compose.yml 
version: "3.9"

services:
  goldengate:
    image: oracle/goldengate:23ai 
    container_name: ogg23ai
    ports:
      - "880:80"
      - "4443:443"
    environment:
      - OGG_ADMIN=ggadmin
      - OGG_ADMIN_PWD=<password>
      - OGG_DOMAIN=OGG23
      - OGG_DEPLOYMENT=OGG23ai
      - SSL_SKIP_CERT_VERIFY=true
    volumes:
      - ./u02:/u02
      - ./u03:/u03 
      - ./cert:/etc/nginx/cert
    restart: unless-stopped
```

``` wp-block-code
[root@docker docker23c]# docker compose up -d
WARN[0000] /u01/docker_files/docker23c/docker-compose.yml: `version` is obsolete 
[+] Running 2/2
 ✔ Network docker23c_default  Created                                                                                                                         0.2s 
 ✔ Container ogg23ai          Started                                                                                                                         1.1s 
[root@docker docker23c]# docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS                            PORTS                                                                          NAMES
4e4c50dfc277   oracle/goldengate:23ai   "/usr/local/bin/depl…"   7 seconds ago   Up 5 seconds (health: starting)   0.0.0.0:880->80/tcp, :::880->80/tcp, 0.0.0.0:4443->443/tcp, :::4443->443/tcp   ogg23ai
```

This will start Oracle GoldenGate Microservices (Admin Server, Distribution Server, Receiver Server, and Performance Metrics Server) inside a container.

### **📋** **Important Notes:**

- Ensure your machine has **enough memory and CPU** (minimum 4GB RAM recommended).
- By default, SSL certificates are created inside the container for HTTPS access.
- You can mount external volumes for persistence if needed.

### **📚** **Resources**

- 🔗 [Oracle Docker Images GitHub](https://github.com/oracle/docker-images)
- 📄 [Oracle GoldenGate Documentation](https://docs.oracle.com/en/middleware/goldengate/index.html)
- 🐳 Docker Installation Guide

### **✨** **Conclusion**

Building your own Oracle GoldenGate 23ai Docker image gives you **flexibility**, **portability**, and **speed** when working on real-time data replication projects.

Using Oracle's provided GitHub build scripts makes it much easier to maintain version control and automation!
