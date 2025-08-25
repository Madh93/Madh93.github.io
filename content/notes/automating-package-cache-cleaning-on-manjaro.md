---
title: Automating Package Cache Cleaning on Manjaro Linux
date: 2025-08-25
summary: Install `paccache-hook` and `yaycache-hook` from the AUR to automatically clean your official and AUR package caches after every transaction, keeping disk usage in check.
categories: [Manjaro, Pacman, Yay, AUR, System Administration]
---

{{< admonition type="tip" title="TL;DR" >}}
Install `paccache-hook` and `yaycache-hook` from the AUR to automatically clean your official and AUR package caches after every `pacman` or `yay` transaction, preventing your cache from growing indefinitely.
{{< /admonition >}}

On Arch-based systems like Manjaro, every package you install or upgrade is stored in a cache directory at `/var/cache/pacman/pkg/`. While this is useful for downgrading or reinstalling packages without re-downloading them, it can cause your disk usage to grow indefinitely if left unmanaged.

The standard tool for cleaning this cache is [`paccache`](https://man.archlinux.org/man/paccache.8), but running it manually can be easy to forget. A much better approach is to automate the process.

## Pacman Hooks

The ideal "set it and forget it" solution is to use [Pacman hooks](https://wiki.archlinux.org/title/Pacman#Hooks), scripts that run automatically after package transactions. The [Arch User Repository (AUR)](https://aur.archlinux.org/) provides two excellent hook packages for this purpose:

* **`paccache-hook`**: For cleaning the cache of official repository packages.
* **`yaycache-hook`**: For cleaning the cache of AUR packages managed by `yay`.

You can install both with a single command:

```shell
yay -S paccache-hook yaycache-hook
```

## How It Works

Once installed, these packages place hook files in `/usr/share/libalpm/hooks/` that are triggered after every `pacman` or `yay` operation.

* **`paccache-hook`** automatically runs `paccache -r`, which by default removes all cached versions of your installed packages except for the **three most recent ones**.
* **`yaycache-hook`** does the exact same thing for the AUR packages that `yay` downloads and builds.

This combination ensures that both your official and AUR package caches are kept tidy automatically. You get the benefit of having recent package versions available for downgrades without letting the cache consume gigabytes of disk space over time.

For more granular control, you can adjust settings like the number of package versions to keep by editing the configuration files at `/etc/paccache-hook.conf` and `/etc/yaycache-hook.conf`.
