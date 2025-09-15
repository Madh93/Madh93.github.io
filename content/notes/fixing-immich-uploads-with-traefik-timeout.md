---
title: Fixing Long Immich Uploads Failing Behind Traefik
date: 2025-09-15
summary: "Fix long video uploads to Immich failing at the 60-second mark when using Traefik by setting the entrypoint's `readTimeout` to '0s', which reverts a recent breaking change in Traefik's default configuration."
categories: [Immich, Traefik, Docker, Self-Hosting, Troubleshooting]
---

{{< admonition type="tip" title="TL;DR" >}}
Fix long video uploads to Immich failing at the 60-second mark when using Traefik by setting the entrypoint's `readTimeout` to '0s', which reverts a recent breaking change in Traefik's default configuration.
{{< /admonition >}}

## The Problem: Large Video Uploads Suddenly Failing

I recently noticed that longer videos from my phone would consistently fail to sync to my self-hosted [**Immich**](https://immich.app/) instance. The upload would start, run for exactly one minute, and then silently fail.

After finding this [detailed GitHub discussion](https://github.com/immich-app/immich/discussions/8872), I realized the problem wasn't Immich itself. The Traefik access logs confirmed that requests to Immich's upload endpoint were being terminated at precisely **60000ms**, pointing directly to a timeout at the reverse proxy level.

## The Culprit: A Traefik Default Timeout Change

The root cause is a [**breaking change**](https://doc.traefik.io/traefik/reference/install-configuration/entrypoints/#transport-respondingTimeouts-readTimeout) introduced in [**Traefik**](https://traefik.io/) **v2.11.2** (and continued in v3). In these versions, the default `readTimeout` for all entrypoints was changed from `0` (meaning no timeout) to a strict **60 seconds**.

This means that any HTTP request that takes longer than 60 seconds to complete—like uploading a large video file—is now automatically terminated by Traefik. Because this change was introduced in a minor version update, it caught many users by surprise when their applications suddenly started timing out.

## The Solution: Explicitly Set the Timeout

To fix this, you need to override the new default by explicitly setting the `readTimeout` in your Traefik static configuration (e.g., `traefik.yml` or command-line arguments).

Setting the value back to `"0s"` restores the old behavior of having no timeout.

```yaml
# In your traefik.yml or static configuration

entryPoints:
  websecure: # Or whichever entrypoint Immich uses
    # ... your other entrypoint config ...
    transport:
      respondingTimeouts:
        readTimeout: "0s" # Reverts to the old behavior (no timeout)
```

After adding this configuration and restarting your Traefik container, the 60-second timeout will be removed, and your long video uploads to Immich will succeed once again.
