---
title: Fixing Cross-Account Route 53 Private Hosted Zone Resolution
date: 2025-10-22
summary: "Learned that for an EC2 instance to resolve a Route 53 private hosted zone from another AWS account, you must create a cross-account VPC association. VPC peering alone is not enough; it's a two-step authorization process."
categories: [AWS, Route53, VPC, Cross-Account, Peering]
---

{{< admonition type="tip" title="TL;DR" >}}
Learned that for an EC2 instance to resolve a [**Route 53**](https://aws.amazon.com/route53/) private hosted zone from another AWS account, you must create a cross-account VPC association. **VPC Peering alone is not enough**; it's a two-step authorization process.
{{< /admonition >}}

I was troubleshooting an [**EC2 instance**](https://aws.amazon.com/ec2/) that was failing to connect to an internal service. After some digging, I realized that when the instance tried to resolve the service's DNS name (e.g., `service.internal.mycompany.com`), it was receiving a **public** IP address from the public hosted zone, not the **private** IP from our Route 53 private hosted zone.

## The Problem

My first assumption was that since the two AWS accounts (**Account A** holding the private zone, **Account B** holding the EC2's VPC) were connected via **VPC Peering**, DNS resolution would just work.

This was incorrect! I learned that network connectivity (peering) and DNS resolution (zone association) are two separate configurations. For the VPC in **Account B** to use the private zone from **Account A**, it must be **explicitly associated** with it.

Because they are in different accounts, this requires a special two-step authorization process, as detailed in [this AWS knowledge center article](https://repost.aws/knowledge-center/route53-private-hosted-zone).

## The Solution

### 1. Authorize the Association in Account A (Hosted Zone Owner)

First, you must log in to the account that owns the private hosted zone (Account A) and explicitly grant permission to the other account's VPC to associate with it.

```shell
aws route53 create-vpc-association-authorization \
  --region 'us-east-1' \
  --hosted-zone-id '<zone-id-from-account-A>' \
  --vpc 'VPCRegion=<vpc-region-of-account-B>,VPCId=<vpc-id-from-account-B>'
```

### 2. Create the Association in Account B (VPC Owner)

Once permission is granted, you log in to the account that owns the VPC (Account B) and run the command to accept the authorization and create the association.

```shell
aws route53 associate-vpc-with-hosted-zone \
  --region 'us-east-1' \
  --hosted-zone-id '<zone-id-from-account-A>' \
  --vpc 'VPCRegion=<vpc-region-of-account-B>,VPCId=<vpc-id-from-account-B>'
```

After a few moments, the association was active, and my EC2 instance in **Account B** immediately started resolving the correct **private IP** addresses from **Account A**'s hosted zone.

### 3. (Optional) Clean Up the Authorization in Account A (Hosted Zone Owner)

After the association is successfully created, it's a security best practice to return to **Account A** and delete the authorization. This prevents the same VPC from being associated again by mistake. The existing, active association will **not** be affected.

```shell
aws route53 delete-vpc-association-authorization \
  --region 'us-east-1' \
  --hosted-zone-id '<zone-id-from-account-A>' \
  --vpc 'VPCRegion=<vpc-region-of-account-B>,VPCId=<vpc-id-from-account-B>'
```
