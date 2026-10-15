---
title: "shell script to take export of docker image docker_image_restore.sh"
author: "Sunke Nithin"
date: 2025-04-28
categories: ["SHELL SCRIPTS"]
tags: []
source: https://nithinsunke.wordpress.com/2025/04/28/shell-script-to-take-export-of-docker-image-docker_image_restore-sh/
---

# shell script to take export of docker image docker_image_restore.sh

``` wp-block-code
#!/bin/bash

# Variables (edit these)
IMPORT_FILE="goldengate23ai.tar.gz"

# Check if file is compressed
if [[ "$IMPORT_FILE" == *.gz ]]; then
    echo " Uncompressing $IMPORT_FILE ..."
    gunzip -f $IMPORT_FILE
    IMPORT_FILE="${IMPORT_FILE%.gz}"
fi

# Load the image
echo " Loading Docker image from $IMPORT_FILE ..."
docker load -i $IMPORT_FILE

if [ $? -eq 0 ]; then
    echo " Docker image loaded successfully."
else
    echo " Failed to load Docker image."
    exit 1
fi

echo " Done!"
```

``` wp-block-preformatted
```
