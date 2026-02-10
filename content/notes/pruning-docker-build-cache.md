---
title: Pruning Docker Build Cache
date: 2026-02-10
summary: "Use `docker builder prune --al` to reclaim GBs of disk space by removing unused build caches, especially effective in long-term development environments."
categories: [Docker, CLI, System Administration]
---

{{< admonition type="tip" title="TL;DR" >}}
Use `docker builder prune --al` to reclaim GBs of disk space by removing unused build caches, especially effective in long-term development environments.
{{< /admonition >}}

Over time, [**Docker**](https://www.docker.com/) build caches can accumulate and consume a significant amount of disk space. Every time you build an image, Docker stores the layers to speed up future builds. While this is great for performance, these caches often persist even after the images they belong to are deleted or updated.

On my work laptop, I recently noticed my storage was getting tight. Upon investigation, I found that the Docker build cache was the main culprit.

## The Command

While a simple `docker system prune` removes stopped containers and dangling images, it often leaves behind a lot of build-related data. To truly clean up the build cache, you should use the dedicated builder command:

```shell
docker builder prune --all
```

{{< admonition type="warning" >}}
Keep in mind that pruning the build cache means your next `docker build` will take longer, as Docker will have to pull base images and rebuild every layer from scratch.
{{< /admonition >}}

The `--all` flag is crucial. Without it, Docker only removes "dangling" build cache (cache from failed builds or those no longer associated with a tagged image). With `--all`, it removes all unused cache, which is where the real space savings are found.

## Results

After running this command on my machine, the results were impressive:

- **Total reclaimed space:** Over **50 GB**.
- **Items deleted:** More than **1,500 build caches**.

This is particularly useful if you work on many different projects or frequently change base images and dependencies in your Dockerfiles.

## Extra Tip: Filtering


If you want to be slightly less aggressive, you can add the `--filter` flag to only remove cache older than a certain duration:

```shell
# Remove build cache older than 48 hours
docker builder prune --all --filter "until=48h"
```

This balances the need for disk space with the benefits of keeping recent cache for your active projects.
