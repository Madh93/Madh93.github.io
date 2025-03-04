---
title: S3 Native State Locking in Terraform
date: 2025-03-01
categories: [Terraform, AWS, S3, Infrastructure-as-Code]
---

{{< admonition type="tip" title="TL;DR" >}}
Terraform 1.11 introduces [S3-native state locking](https://developer.hashicorp.com/terraform/language/backend/s3#state-locking), eliminating the need for DynamoDB-based locks.
{{< /admonition >}}

Traditionally, [Terraform](https://www.terraform.io/) used **DynamoDB-based locking** to prevent concurrent state modifications when using an [S3 backend](https://developer.hashicorp.com/terraform/language/backend/s3). With [Terraform 1.11](https://www.hashicorp.com/en/blog/terraform-1-11-ephemeral-values-managed-resources-write-only-arguments), S3-native state locking is now generally available, allowing users to manage state locks directly through S3 without relying on DynamoDB.

This change simplifies state management by reducing dependencies and improving performance. While you can still use DynamoDB for migration purposes, Terraform **recommends** migrating to the new S3-native state locking mechanism.

## Enabling S3 Native State Locking

To enable S3-native state locking, update your backend configuration to include the `use_lockfile` argument:

```hcl
terraform {
  backend "s3" {
    bucket       = "my-terraform-state-bucket"
    key          = "state/terraform.tfstate"
    region       = "us-east-1"    
    use_lockfile = true # Enables S3-native state locking
  }
}
```

Once enabled, Terraform will automatically handle state locking via {{< sidenote "S3" >}} Don't forget to update the required IAM permissions.{{< /sidenote >}}, preventing simultaneous operations on the state file.

{{< admonition >}}
If you're currently using DynamoDB for state locking, **you can enable both S3 and DynamoDB locking simultaneously** to ensure a smooth transition. However, DynamoDB-based locking will be removed in a future minor release, so it's best to migrate fully to S3-native locking as soon as possible.
{{< /admonition >}}
