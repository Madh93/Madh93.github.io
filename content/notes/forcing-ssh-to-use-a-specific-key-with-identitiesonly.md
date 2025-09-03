---
title: "Forcing SSH to Use a Specific Key with IdentitiesOnly"
date: 2025-09-03
summary: "Use `IdentitiesOnly=yes` in your SSH config to force the client to use only the specified key, avoiding 'Too many authentication failures' errors when you have multiple keys."
categories: [SSH, Security, CLI, Networking]
---

{{< admonition type="tip" title="TL;DR" >}}
Use `IdentitiesOnly=yes` in your SSH config to force the client to use only the specified key, avoiding "Too many authentication failures" errors when you have multiple keys.
{{< /admonition >}}

If you manage multiple SSH keys (e.g., for personal projects, work, and different clients), you may have encountered the frustrating error:

```shell
Received disconnect from host: 2: Too many authentication failures
```

## The Root Cause

This happens because, by default, the SSH client offers every key it knows about (from your `~/.ssh` directory or `ssh-agent`) to the server, one by one. Many servers are configured to terminate the connection after a set number of failed attempts (often 5 or 6). If you have more keys than the server's limit, or if the correct key isn't one of the first few tried, the server will kick you out before you can successfully authenticate.

## The Solution

The solution is to tell your SSH client to be more precise. The `IdentitiesOnly=yes` option instructs the client to attempt authentication using **only** the identity files that are explicitly specified for that host, either in your `~/.ssh/config` or via the `-i` flag on the command line.

You can set this in your `~/.ssh/config` on a per-host basis. This ensures that when you connect to a specific host, only the correct key is ever presented.

```ini
# ~/.ssh/config

Host my-work-server
  HostName 123.45.67.89
  User myuser
  IdentityFile ~/.ssh/work_key
  IdentitiesOnly yes

Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
```

## Why Isn't It the Default?

The default behavior (`IdentitiesOnly=no`) is designed for convenience. In a simple setup with just one or two keys, it's very handy that the SSH client can automatically find and use the correct key without you needing to configure every host.

However, as the number of keys you manage grows, this convenience becomes a liability. Switching to `IdentitiesOnly=yes` for your configured hosts provides a more robust and explicit approach that scales much better and avoids unexpected connection failures.
