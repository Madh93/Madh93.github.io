---
title: Viewing Compressed Log Files with Zutils
date: 2025-10-08
summary: Discovered **zutils** like `zcat`, `zgrep`, and `zless` that allow you to read, search, and page through compressed log files directly without needing to manually decompress them first.
categories: [CLI, Linux, Logs, System Administration]
---

{{< admonition type="tip" title="TL;DR" >}}
Discovered **zutils** like `zcat`, `zgrep`, and `zless` that allow you to read, search, and page through compressed log files directly, without needing to manually decompress them first.
{{< /admonition >}}

System logs are often rotated and compressed to save disk space, leaving you with files like `syslog.1.gz`, `app.log.2.gz`, and so on. My old workflow for inspecting these was cumbersome and inefficient:

1. Decompress the file: `gunzip /var/log/app.log.2.gz`
2. Read or search it: `grep "ERROR" /var/log/app.log.2`
3. Re-compress it to clean up: `gzip /var/log/app.log.2`

This multi-step process is slow, requires temporary disk space, and is just plain tedious.

## Zutils

I recently learned about the family of [**zutils**](https://www.nongnu.org/zutils/zutils.html) available on most Linux systems, which are designed specifically for this purpose. These tools are compressed-aware versions of common commands that decompress files on-the-fly into a stream, without ever creating a temporary file on disk.

Some of the most useful ones include:

- **`zcat`**: Acts like `cat`. It reads one or more compressed files and prints their uncompressed contents to standard output.
    ```shell
    # Instead of gunzip -> cat
    zcat /var/log/syslog.1.gz
    ```
- **`zgrep`**: Acts like `grep`. It allows you to search for patterns inside compressed files.
    ```shell
    # Search for a specific error code in all rotated logs
    zgrep "E-1042" /var/log/app.log.*.gz
    ```
- **`zless`**: Acts like `less`. This is often the most practical, as it lets you interactively view and scroll through large compressed log files.
    ```shell
    # Page through a large compressed log file
    zless /var/log/nginx/access.log.3.gz
    ```

## Conclusion

Using these utilities is a massive improvement. It's a single-step process that is faster, doesn't consume extra disk space, and makes the command history cleaner. For anyone who regularly works with log files on a server, adopting the **zutils** is a fundamental time-saver.
