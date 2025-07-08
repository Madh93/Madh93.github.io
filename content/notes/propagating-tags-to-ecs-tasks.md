---
title: Propagating Tags to AWS ECS Tasks
date: 2025-07-08
summary: By default, user-defined tags on an AWS ECS Service are not propagated to its Tasks, which can impact cost allocation. This can be fixed by setting the propagate_tags parameter.
categories: [AWS, ECS, Cost Management, Terraform]
---

{{< admonition type="tip" title="TL;DR" >}}
By default, user-defined tags on an AWS ECS Service are not propagated to its Tasks, which can impact cost allocation. This can be fixed by setting the `propagate_tags` parameter.
{{< /admonition >}}

If you use [AWS tags on your resources for cost allocation](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html), you might run into a surprise with Amazon ECS. I discovered that when you assign **user-defined tags** to an ECS Service, those tags are **not automatically propagated** to the Tasks that the service launches.

This is a significant issue because if your cost tracking reports rely on these tags, the costs associated with your ECS Tasks may not be correctly attributed, leading to unexpected expenses showing up as "untagged."

## The Root Cause

By default, ECS automatically applies its own **managed tags** (like the cluster name) to all newly launched tasks, but it does **not** do the same for the custom, user-defined tags you set on the service level.

The propagation of tags from a service or task definition to a task is controlled by a specific parameter that defaults to `NONE`. This means you have to explicitly enable this behavior.

## The Solution

To ensure your user-defined tags are correctly passed down from the service to the tasks, you must enable tag propagation.

### Using AWS CLI

When [updating an existing service](https://docs.aws.amazon.com/cli/latest/reference/ecs/update-service.html), you can force a new deployment and enable tag propagation using the `--propagate-tags` flag.

```shell
aws ecs update-service \
  --cluster <CLUSTER_NAME> \
  --service <SERVICE_NAME> \
  --force-new-deployment \
  --propagate-tags SERVICE
```

The valid values are `SERVICE` or `TASK_DEFINITION`, depending on where your source tags are defined.

### Using Terraform

If you manage your infrastructure with Terraform, the fix is straightforward. You just need to set the `propagate_tags` argument within your `aws_ecs_service` [resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_service) block:

```terraform
resource "aws_ecs_service" "my_service" {
  # ... other configuration
  
  propagate_tags = "SERVICE" # or "TASK_DEFINITION"
}
```

By enabling this setting, you ensure that your cost allocation tags are correctly applied, giving you an accurate picture of your ECS spending.
