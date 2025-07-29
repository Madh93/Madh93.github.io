---
title: Creating a Random Delay for Staggered Operations with shuf
date: 2025-07-29
summary: Use 'shuf -i MIN-MAX -n 1' to generate a random number, perfect for creating a staggered delay with the 'sleep' command in deployment scripts.
categories: [shuf, Linux, Shell, CLI, Automation]
---

{{< admonition type="tip" title="TL;DR" >}}
Use `shuf -i MIN-MAX -n 1` to generate a random number, perfect for creating a staggered delay with the `sleep` command in deployment scripts.
{{< /admonition >}}

When running a command across multiple servers at once (for example, with an Ansible playbook or a simple SSH loop), you often want to avoid having all machines execute a critical action at the exact same moment. Restarting a service like `php-fpm` or `nginx` simultaneously on every node could lead to a brief but complete service outage.

A simple and elegant way to prevent this is to introduce a random delay at the beginning of your command. The GNU Core Utilities provide a perfect tool for this: [`shuf`](https://man7.org/linux/man-pages/man1/shuf.1.html).

## The `shuf` Command

The `shuf` command is used to generate random permutations of its input. When used with the `-i` flag, it can select random numbers from a given integer range.

- `shuf`: The command itself.
- `-i MIN-MAX`: Specifies the input range of integers. For example, `-i 0-60` provides numbers from 0 to 60.
- `-n COUNT`: Tells `shuf` how many numbers to output. For our use case, we just need one (`-n 1`).

So, `shuf -i 0-60 -n 1` will print a single random integer between 0 and 60.

## Use Case

You can combine `shuf` with the `sleep` command to pause a script for a random duration. This is extremely useful for staggering deployments.

```shell
sleep $(shuf -i 0-60 -n 1); sudo systemctl restart php-fpm && sudo systemctl reload nginx
```

When this command is executed across a fleet of servers, each server will wait for a different random amount of time (between 0 and 60 seconds) before restarting its services. This effectively spreads the restarts over a one-minute window, preventing them from all happening at once.

This is a powerful, dependency-free technique to de-synchronize actions and improve the resilience of your automated tasks.
