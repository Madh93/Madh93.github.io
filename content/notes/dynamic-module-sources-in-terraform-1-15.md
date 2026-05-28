---
title: Dynamic Module Sources in Terraform 1.15
date: 2026-05-28
summary: Terraform 1.15 introduces the `const = true` attribute for variables, allowing them to be evaluated at `terraform init` time. This means you can finally use variables inside a module's `source` and `version` attributes.
categories: [Terraform, Infrastructure-as-Code, Development]
---

{{< admonition type="tip" title="TL;DR" >}}
Terraform 1.15 introduces the `const = true` attribute for variables, allowing them to be evaluated at `terraform init` time. This means you can finally use variables inside a module's `source` and `version` attributes.
{{< /admonition >}}

For years, defining a module in Terraform came with a strict limitation: the `source` and `version` attributes had to be hardcoded string literals. You could not drive them from variables or locals. 

In practice, this forced developers into bad architectural patterns, such as copy-pasting identical module blocks across different environments just to change the `ref`, or creating thin, useless wrapper modules.

## The Monorepo Problem

This limitation was especially painful if you used a monorepo for your Terraform modules. Because the version couldn't be parameterized, it was incredibly easy for different modules within the same project to drift into a messy state of mixed versions. 

For example, a single `main.tf` file would end up looking like this:

```hcl
module "config" {
  source = "git@bitbucket.org:company/org-infra-tfmodules//utils-config?ref=3.21.0"
}

module "kms" {
  source = "git@bitbucket.org:company/org-infra-tfmodules//aws-kms?ref=3.2.0"
}

module "api_gateway" {
  source = "git@bitbucket.org:company/org-infra-tfmodules//aws-api-gateway?ref=3.17.2"
}
```

Updating the infrastructure meant manually tracking down and bumping dozens of independent string literals, inevitably leading to missed updates and inconsistency.

## The Fix: Constant Variables

Terraform 1.15 solves this by introducing a new `const` attribute for variables. A constant variable is evaluated during the `terraform init` phase—before any providers or resources are evaluated—making it completely legal to use inside `source` and `version` blocks.

To fix the monorepo drift, we can now define a single constant variable for our infrastructure modules version:

```hcl
variable "tfmodules_repo" {
  type    = string
  const   = true
  default = "git@bitbucket.org:company/org-infra-tfmodules"
}

variable "tfmodules_version" {
  type    = string
  const   = true
  default = "3.25.0"
}
```

And then dynamically interpolate this variable across all our module sources:

```hcl
module "config" {
  source = "${var.tfmodules_repo}//utils-config?ref=${var.tfmodules_version}"
}

module "kms" {
  source = "${var.tfmodules_repo}//aws-kms?ref=${var.tfmodules_version}"
}

module "api_gateway" {
  source = "${var.tfmodules_repo}//aws-api-gateway?ref=${var.tfmodules_version}"
}
```

Now, updating the entire suite of modules to a new release is as simple as bumping the default value of `tfmodules_version` or passing it via the CLI during initialization.

{{< admonition type="note" >}}
Because `const` variables **are required during initialization**, operations that load the configuration (like `terraform plan`, `show`, `test`, or `validate`) will also require these variables to be passed if no default is set.
{{< /admonition >}}
