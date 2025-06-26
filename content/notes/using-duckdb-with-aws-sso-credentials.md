---
title: Using DuckDB with AWS SSO Credentials
date: 2025-06-26
summary: When using DuckDB with AWS SSO, credential errors can occur when accessing S3. A shell function that wraps duckdb with aws configure export-credentials provides a seamless workaround.
categories: [DuckDB, AWS, SSO, CLI, Shell]
---

{{< admonition type="tip" title="TL;DR" >}}
When using [DuckDB](https://duckdb.org/) with [AWS SSO](https://aws.amazon.com/single-sign-on/), credential errors can occur when accessing S3. A shell function that wraps `duckdb` with [`aws configure export-credentials`](https://docs.aws.amazon.com/cli/latest/reference/configure/export-credentials.html) provides a seamless workaround.
{{< /admonition >}}

My colleague [Raúl](https://in4matics.cat/) recently showed me the advantages of [DuckDB](https://duckdb.org/), a powerful analytical database that can directly query data from Parquet, CSV, and JSON files, including those stored in S3. However, I ran into a problem when trying to use it with credentials managed by [AWS SSO](https://aws.amazon.com/single-sign-on/).

### The Problem

The DuckDB AWS extension does not natively support the AWS SSO authentication flow. This results in credential errors when you try to query a file directly from an S3 bucket. There are several [open GitHub issues](https://github.com/duckdb/duckdb-aws/issues/62) detailing this problem.

### The Workaround

After reading through the comments on the issue, I found a clean and effective workaround for Linux/macOS users running `bash` or `zsh`. It involves creating a shell function that wraps the `duckdb` command.

You can add the following function to your shell configuration file (e.g., `~/.bashrc` or `~/.zshrc`):

```bash
function duckdb() {
  (eval "$(aws configure export-credentials --format env)" && command duckdb "$@")
}
```

### How It Works

This simple function makes the integration seamless:

1. **`function duckdb() { ... }`**: It creates a new function named `duckdb`. When you type `duckdb` in your terminal, this function is called instead of the original binary.
2. **`aws configure export-credentials --format env`**: This command fetches temporary, short-lived credentials (access key, secret key, and session token) from your active AWS SSO session and outputs them in a format that can be used to set environment variables.
3. **`eval "$(...)"`**: This executes the output from the `aws` command, setting the `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_SESSION_TOKEN` environment variables.
4. **`( ... )`**: The entire command is wrapped in a subshell. This is a key benefit, as it ensures the exported credentials only exist for the scope of this single command and do not leak into your main shell session.
5. **`command duckdb "$@"`**: Finally, it executes the original `duckdb` command, passing along any arguments you provided (`$@`). The DuckDB AWS extension automatically picks up the standard AWS environment variables for authentication.

After reloading your shell (`source ~/.zshrc`), you can now run commands transparently as you normally would, and they will work perfectly with S3:

```shell
duckdb -c "SELECT * FROM 's3://somebucket/somefile.csv'"
```
