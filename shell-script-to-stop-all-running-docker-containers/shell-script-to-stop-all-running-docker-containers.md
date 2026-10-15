---
title: "Shell script to stop all running docker containers."
author: "Sunke Nithin"
date: 2025-08-12
categories: ["SHELL SCRIPTS"]
tags: []
source: https://nithinsunke.wordpress.com/2025/08/12/shell-script-to-stop-all-running-docker-containers/
---

# Shell script to stop all running docker containers.

``` wp-block-code
#!/bin/bash

# List all running Docker containers
RUNNING_CONTAINERS=$(docker ps --format '{{.Names}}')

# Check if there are any running containers
if [ -z "$RUNNING_CONTAINERS" ]; then
    echo "No running Docker containers found."
    exit 0
fi

# Display the names of running containers
echo "Running Docker containers:"
echo "$RUNNING_CONTAINERS"

# Stop all running containers
echo "Stopping all running Docker containers..."
docker stop $RUNNING_CONTAINERS

echo "All running Docker containers have been stopped."
```
