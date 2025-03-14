---
title: Generating Least Privilege IAM Policies using iamlive
date: 2025-03-14
categories: [AWS, IAM, Terraform]
---

{{< admonition type="tip" title="TL;DR" >}}
[iamlive](https://github.com/iann0036/iamlive) can automatically generate least privilege IAM policies based on the permissions used during Terraform execution.
{{< /admonition >}}

Today I learned about [iamlive](https://github.com/iann0036/iamlive), a tool that generates IAM policies following the principle of least privilege. I specifically needed it for [Terraform](https://www.terraform.io/) to ensure that the IAM role I use to deploy my resources only gets the permissions it truly needs, reducing security risks in my AWS environment.

## How It Works

When you run [Terraform](https://www.terraform.io/) commands (like `plan` or `apply`), `iamlive` intercepts the API calls made by Terraform. It analyzes the actions required and generates a corresponding IAM policy, ensuring that your policies are as tight as possible.

### Running iamlive

In one terminal, you start `iamlive` in proxy mode and specify an output file for the generated policy:

```shell
iamlive --mode proxy --output-file policy.json
```

### Integrating with Terraform

In another terminal, configure your environment to route Terraform's API calls through `iamlive` by setting up your proxy and certificate bundle:

```shell
terraform init
export HTTP_PROXY=http://127.0.0.1:10080
export HTTPS_PROXY=http://127.0.0.1:10080
export AWS_CA_BUNDLE=~/.iamlive/ca.pem
terraform plan
terraform apply
```

These commands initialize Terraform, set the necessary environment variables to direct API calls through `iamlive`, and then run your usual Terraform `plan` and `apply` steps.

As you execute these commands, `iamlive` observes the required permissions and generates a policy accordingly 😃
