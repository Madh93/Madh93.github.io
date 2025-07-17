---
title: Listing Resources and Data Sources from a Terraform Provider
date: 2025-07-14
summary: Use `terraform providers schema -json` and `jq` to extract a complete list of all available resources and data sources for any Terraform provider.
categories: [Terraform, AWS, CLI, jq]
---

{{< admonition type="tip" title="TL;DR" >}}
Use `terraform providers schema -json` and `jq` to extract a complete list of all available resources and data sources for any Terraform provider.
{{< /admonition >}}

For scripting, automation, or simply to explore what's available, it's incredibly useful to have a complete list of all the resources and data sources that a Terraform provider offers. I learned that you can generate this list directly from the [Terraform CLI](https://developer.hashicorp.com/terraform/cli/commands/providers/schema) with a little help from the command-line JSON processor, `jq`.

## Setup

First, you need an initialized Terraform directory with the provider you want to inspect. This allows Terraform to load the provider's schema, which contains all the information we need.

1. **Create a `main.tf` file** with the provider definition. You don't need any other configuration.

    ```hcl
    terraform {
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.100.0"
        }
      }
    }
    ```

2. **Initialize Terraform** in that directory.

    ```shell
    terraform init
    ```

## Listing Resources and Data Sources

With the provider initialized, you can now dump its schema as JSON and use `jq` to parse it.

### Resources

```shell
terraform providers schema -json | jq -r '.provider_schemas."registry.terraform.io/hashicorp/aws".resource_schemas | keys[]'
```

### Data Sources

The command is almost identical, you just need to change `resource_schemas` to `data_source_schemas`.

```shell
terraform providers schema -json | jq -r '.provider_schemas."registry.terraform.io/hashicorp/aws".data_source_schemas | keys[]'
```

You can easily adapt this for any provider by changing the path `"registry.terraform.io/hashicorp/aws"` to match the provider you are inspecting.
