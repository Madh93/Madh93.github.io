---
title: "Fixing the 'zsh: corrupt history file' Error"
date: 2025-09-10
summary: "Fix the 'zsh: corrupt history file' error by backing up the file, filtering out non-printable characters with the 'strings' command, and reloading the cleaned history with 'fc -R'."
categories: [ZSH, Shell, CLI, Linux, Troubleshooting]
---

{{< admonition type="tip" title="TL;DR" >}}
Fix the `zsh: corrupt history file` error by backing up the file, filtering out non-printable characters with the `strings` command, and reloading the cleaned history with `fc -R`.
{{< /admonition >}}

## The Problem

Out of nowhere, my terminal started greeting me with the `zsh: corrupt history file` error upon opening a new session. This annoying message means your command history file, typically located at `~/.zsh_history`, contains invalid or non-printable characters that [`zsh`](https://es.wikipedia.org/wiki/Zsh) cannot parse.

This corruption usually happens due to an improper shutdown, a system crash, or sometimes when multiple concurrent terminal sessions write to the file at the same time, causing a conflict.

## The Solution

Instead of deleting the file and losing your valuable command history, you can clean it and reload it. This process involves backing up the bad file, using the [`strings`](https://linux.die.net/man/1/strings) utility to strip out the corrupting binary characters, and then forcing `zsh` to read the newly cleaned file.

The following commands accomplish this safely:

```shell
# 1. Move the corrupted history file to a backup location
mv ~/.zsh_history ~/.zsh_history_bad

# 2. Filter out the corrupt binary data and save the clean output
strings ~/.zsh_history_bad > ~/.zsh_history

# 3. Read the clean history file into your current session's memory
fc -R ~/.zsh_history
```

After running these commands, the error message will be gone, and your command history will be restored. Opening a new terminal tab or window will confirm the fix is persistent.
