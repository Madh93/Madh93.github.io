---
title: Why Steampipe is My Go-To Tool for Querying Cloud APIs
date: 2025-10-17
summary: Steampipe is a tool I've used for ages that maps APIs to a PostgreSQL database, letting you query them with SQL. Its killer feature is using connection aggregators to query multiple AWS accounts and regions at once, replacing complex shell scripts.
categories: [Steampipe, CLI, SQL, AWS, Jira]
---

## What is Steampipe?

I've been using a tool for a long time that has become indispensable in my daily work: [**Steampipe**](https://steampipe.io/). Its premise is simple but incredibly powerful: it exposes hundreds of external APIs as tables in a PostgreSQL database.

This means you can use standard SQL to query, join, and analyze data from all your essential services. You can query your cloud resources, CI/CD pipelines, version control systems, and even ticketing systems, all from one place.

## The Plugin Ecosystem

Steampipe's magic comes from its vast plugin ecosystem. Each service (like AWS, Azure, GitHub, or Jira) has its own plugin that you install with a simple command:

```shell
# Install the AWS and Jira plugins
steampipe plugin install aws
steampipe plugin install jira
```

You can find a massive list of available plugins for almost any service you can think of on the [**Steampipe Hub**](https://hub.steampipe.io/?engines=steampipe).

Once installed, you add your credentials in a simple `.spc` file, and Steampipe handles all the API calls, pagination, and data mapping for you. You just write the SQL.

## My Use Cases

### AWS Multi-Account & Multi-Region Queries

My primary use case, and where Steampipe truly shines, is as a powerful alternative to the AWS CLI. When I need to find a specific resource (like an old Lambda runtime or a non-compliant S3 bucket) across **all** of our company's AWS accounts and regions, the CLI is a nightmare.

The traditional method is a messy `bash` script that loops through a list of AWS profiles, then nests another loop for a list of regions, runs the `aws` command, and tries to patch all the JSON outputs together using `jq`. It's slow, error-prone, and hard to maintain.

Steampipe solves this with **connection aggregators**. You define all your individual AWS accounts (profiles) and their target regions **once** in your configuration.

Here is a sanitized version of my `aws.spc` configuration:

```ini
# ~/.steampipe/config/aws.spc

# 1. Create the main aggregator connection.
# This connection bundles all other connections
# that match the wildcard "aws_*"
connection "aws" {
  plugin      = "aws"
  type        = "aggregator"
  connections = ["aws_*"]
}

# 2. Define the individual account connections
# Steampipe will query all specified regions for each profile.
connection "aws_prod" {
  plugin  = "aws"
  profile = "production"
  regions = ["eu-west-1", "us-east-1", "eu-central-1"]
}

connection "aws_staging" {
  plugin  = "aws"
  profile = "staging"
  regions = ["eu-west-1", "us-east-1"]
}

connection "aws_security" {
  plugin  = "aws"
  profile = "security"
  regions = ["us-east-1"]
}
```

With this config in place, I can find every Lambda function using an old `python3.8` runtime across **all** configured accounts and regions with **one command**:

```shell
steampipe query "SELECT account_id, region, name, runtime FROM aws.aws_lambda_function WHERE runtime = 'python3.8'"
```

The output is a single, clean SQL table, with Steampipe handling all the concurrent API calls, authentication, and aggregation in the background.

```text
+--------------+-----------+------------------------------+-----------+
| account_id   | region    | name                         | runtime   |
+--------------+-----------+------------------------------+-----------+
| 111122223333 | eu-west-1 | prod-billing-processor       | python3.8 |
| 111122223333 | us-east-1 | prod-legacy-report-generator | python3.8 |
| 444455556666 | eu-west-1 | staging-user-service-cleanup | python3.8 |
| 777788889999 | us-east-1 | security-cw-alerts-to-slack  | python3.8 |
+--------------+-----------+------------------------------+-----------+
```

This is a complete game-changer for cloud inventory and security auditing. No more bash scripts, just simple SQL.

### Querying Jira Issues

But Steampipe isn't just for AWS. Its power is in its breadth. For example, I also use it to query [**Jira**](https://www.atlassian.com/software/jira).

First, the configuration:

```ini
# ~/.steampipe/config/jira.spc

connection "jira" {
  plugin = "jira"

  # The baseUrl of your Jira Instance API
  base_url = "https://my-company.atlassian.net/"

  # The user name to access the jira cloud instance
  username = "my.name@my-company.com"

  # Access Token for which to use for the API
  token = "secret_token"
}
```

With this configured, I can quickly pull my top 5 assigned tickets for a specific project, directly from the command line. I can even change the output format.

```shell
steampipe query "SELECT key, status, summary FROM jira_issue WHERE (project_key = 'SRE') and (assignee_display_name = 'My Name') ORDER BY created DESC LIMIT 5" --output csv
```

The result is a clean CSV, perfect for reports or piping to other tools:

```csv
key,status,summary
SRE-1234,In Progress,Deploy new Lambda runtime to production
SRE-1230,Code Review,Disable legacy SSM job
SRE-1225,Closed,Install php8.4 on new web server
SRE-1224,Closed,Configure web app for php8.4
SRE-1222,Closed,Install php8.4 on staging server
```

## Conclusion

Steampipe has genuinely changed how I approach cloud inventory, auditing, and even simple daily tasks. The ability to use a single, unified language (SQL) to interact with dozens of different services is incredibly efficient.

If you find yourself writing complex scripts to wrangle JSON from different APIs, I highly recommend giving it a try.
