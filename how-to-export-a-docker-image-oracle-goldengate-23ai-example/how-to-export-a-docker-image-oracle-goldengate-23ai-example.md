---
title: "How to Export a Docker Image: Oracle GoldenGate 23ai Example"
author: "Sunke Nithin"
date: 2025-04-28
categories: ["Goldengate", "Virtualization"]
tags: ["CLOUD", "devops", "DOCKER", "python", "technology"]
source: https://nithinsunke.wordpress.com/2025/04/28/how-to-export-a-docker-image-oracle-goldengate-23ai-example/
---

# How to Export a Docker Image: Oracle GoldenGate 23ai Example

After spending time building a custom **Docker image** like **Oracle GoldenGate 23ai**, you may want to **export it** to:

- Move it to another server
- Archive it for backup
- Share it without uploading to Docker Hub

In this blog post, we'll go through the steps to **export and import** a Docker image easily.

### **📥** **Step 1: Identify Your Image**

First, verify the Docker image you want to export:

docker images

Sample output:

REPOSITORY               TAG       IMAGE ID       CREATED          SIZE  
oracle/goldengate         23ai      c3b2d7ab9ab4   5 minutes ago    7.3GB

Here, we want to export the image oracle/goldengate:23ai.

### **📤** **Step 2: Save (Export) the Docker Image**

Use the docker save command to **export** the image into a .tar file:

docker save -o goldengate23ai.tar oracle/goldengate:23ai

Explanation:

- -o goldengate23ai.tar : Output file name.
- oracle/goldengate:23ai : Name and tag of the image to export.

This command will create a **tar archive** of your entire image, including all layers and metadata.

### **📦** **Step 3: (Optional) Compress the Image**

Since Docker images can be large (especially GoldenGate images!), you might want to **compress** it:

gzip goldengate23ai.tar

It will create a file like:

goldengate23ai.tar.gz

Much easier to copy or move!

### **🚚** **Step 4: Move or Share the File**

Now you can:

- Copy it to another server using scp or rsync
- Upload to a private storage bucket (OCI, S3, etc.)
- Store it in a NAS or external drive

### **📥** **Step 5: Load (Import) the Docker Image on Another Machine**

On the target machine where you want to import the image:

1.  First copy the .tar file to the new machine.
2.  Then **load** it back into Docker:

docker load -i goldengate23ai.tar

✅ Now the image is ready to use! You can verify with:

docker images

You should see your oracle/goldengate:23ai listed.

### **🔥** **Bonus Tip: Tag and Push to Private Registry**

Instead of manually moving .tar files every time, you can also:

- Set up a **private Docker registry**.
- Push and pull images securely over your network.

docker tag oracle/goldengate:23ai myregistry.local:5000/oracle/goldengate:23ai

docker push myregistry.local:5000/oracle/goldengate:23ai

Exporting Docker images is a simple but powerful technique to make your GoldenGate deployments portable and backup-friendly!

### **✨** **Conclusion**

In real-world environments, it's always smart to **export** your critical Docker images like **Oracle GoldenGate 23ai** — especially after a successful custom build.

It saves time, provides disaster recovery, and makes migrations easier across environments!
