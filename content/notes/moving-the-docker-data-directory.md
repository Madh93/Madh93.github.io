---
title: Moving the Docker Data Directory on Linux
date: 2025-08-13
summary: To free up disk space on your root partition, you can move Docker's data directory by stopping the service, adding a 'data-root' key to /etc/docker/daemon.json, using rsync to copy the data, and then restarting the service.
categories: [Docker, Linux, CLI, System Administration]
---

{{< admonition type="tip" title="TL;DR" >}}
To free up disk space on your root partition, you can move Docker's data directory by stopping the service, adding a `data-root` key to `/etc/docker/daemon.json`, using `rsync` to copy the data, and then restarting the service.
{{< /admonition >}}

Docker's [default data directory](https://docs.docker.com/engine/daemon/#daemon-data-directory), `/var/lib/docker`, can grow very large over time, consuming precious space on the root filesystem. A common solution is to move this directory to a larger, dedicated partition or disk without losing any of your existing images, volumes, or containers.

## The Process

The process is straightforward and involves carefully stopping Docker, configuring the new path, and safely moving the data.

### 1. Stop the Docker Service

First, it's crucial to completely stop the Docker daemon to prevent any data corruption during the move.

```shell
sudo systemctl stop docker.socket
```

### 2. Configure the New Path

Next, tell the Docker daemon where to find its data root. This is done by editing or creating the [configuration file](https://docs.docker.com/engine/daemon/#configuration-file) at `/etc/docker/daemon.json` and adding the `data-root` key.

```json
{
  "data-root": "/path/to/your/new/docker-root"
}
```

### 3. Copy the Data

Now, use `rsync` to copy the data from the old location to the new one. Using `rsync` is important as it preserves permissions and ownership. The trailing slash on the source directory (`/var/lib/docker/`) ensures the *contents* of the directory are copied, not the directory itself.

```shell
sudo rsync -avh --progress /var/lib/docker /path/to/your/new/docker-root
```

### 4. Restart Docker

With the data moved and the configuration in place, you can restart the service.

```shell
sudo systemctl start docker.socket
```

## Verification and Cleanup

Before removing the old data, verify that Docker is working correctly and pointing to the new location.

```shell
docker info | grep "Docker Root Dir"
# Expected output: Docker Root Dir: /path/to/your/new/docker-root

docker run --rm -it hello-world
# Expected output: Hello from Docker!
```

{{< admonition type="warning" >}}
Only after you have confirmed that everything is working perfectly, you can reclaim the disk space by removing the old Docker data directory.
{{< /admonition >}}

Double-check your paths before running this command.

```shell
sudo rm -rf /var/lib/docker
```
