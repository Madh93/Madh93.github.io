---
title: Why Your AWS Policy Shows a Strange ID Instead of an ARN
date: 2025-11-04
summary: "Discovered that AWS resource-based policies show a Unique Principal ID (e.g., 'AIDA...') when the original IAM principal (user/role) has been deleted. This ID is how IAM internally distinguishes principals to prevent issues with name reuse."
categories: [AWS, IAM, Security, Troubleshooting]
---

{{< admonition type="tip" title="TL;DR" >}}
Discovered that AWS resource-based policies show a **Unique Principal ID** (e.g., 'AIDA...') when the original [**IAM**](https://aws.amazon.com/iam/) principal (user/role) has been deleted. This ID is how IAM internally distinguishes principals to prevent issues with name reuse.
{{< /admonition >}}

## A Mysterious Principal in My Policy

I was auditing an old S3 bucket policy and noticed a very strange `Principal` entry. Instead of a familiar ARN, I just saw a random string of characters.

**What I expected to see:**

```json
"Principal": {
  "AWS": "arn:aws:iam::123456789012:user/my-app-user"
}
```

**What I actually saw:**

```json
"Principal": {
  "AWS": "AIDAJQABLZS4A3QDU576Q"
}
```

My first thought was that this was a typo or a different kind of service principal. I had no idea what an `AIDA` prefix meant.

## Unique Principal IDs

After some research, I learned this is an **IAM Unique Principal ID**.

As the [official IAM documentation explains](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html), when IAM creates a principal (like a user or role), it assigns two identifiers:

1. A **Friendly Name** (e.g., `my-app-user`), which we use.
2. A **Unique ID** (e.g., `AIDA...`), which IAM uses internally.

This ID is crucial for security. If you delete an IAM user named `john`, and later hire a new employee also named `john`, you can create a new IAM user with the same friendly name. However, the old and new `john` users will have different unique IDs. This prevents the new `john` from accidentally inheriting permissions from policies that still reference the old, deleted user.

## Why It Appears

According to [this AWS re:Post article](https://repost.aws/knowledge-center/iam-resource-policy-format), the mystery is solved.

When you save a resource-based policy (like an S3 bucket policy or KMS key policy) that contains an IAM user or role ARN, AWS immediately translates that ARN into the principal's **Unique ID** and saves it.

If that original IAM user or role is later **deleted**, AWS can no longer map the Unique ID back to a valid ARN. As a result, when you view the policy, AWS simply displays the raw Unique ID it has on file.

In short: **seeing an `AIDA...` or `AROA...` ID in a policy is a sign that the principal is stale and has been deleted.**

The only fix is to edit the policy and manually remove the stale ID, or replace it with the ARN of a new, valid principal.

## Common ID Prefixes

These IDs have standard prefixes. The most common ones you'll see are `AIDA` for users and `AROA` for roles, but there is a whole list:

| Prefix | Resource type |
| --- | --- |
| `ABIA` | AWS STS service bearer token |
| `ACCA` | Context-specific credential |
| `AGPA` | User group |
| `AIDA` | IAM user |
| `AIPA` | Amazon EC2 instance profile |
| `AKIA` | Access key |
| `ANPA` | Managed policy |
| `ANVA` | Version in a managed policy |
| `APKA` | Public key |
| `AROA` | Role |
| `ASCA` | Certificate |
| `ASIA` | Temporary (AWS STS) access key |
