---
title: Security Group Referencing for AWS Transit Gateway
date: 2025-08-11
summary: AWS Transit Gateway now supports Security Group referencing, allowing you to reference SGs from other VPCs in your inbound rules. This finally removes a key reason for maintaining VPC Peering connections.
categories: [AWS, Transit Gateway, VPC, Peering, Networking]
---

{{< admonition type="tip" title="TL;DR" >}}
[AWS Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) now supports Security Group referencing, allowing you to reference SGs from other VPCs in your inbound rules. This finally removes a key reason for maintaining [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html) connections.
{{< /admonition >}}

Until today, I was maintaining some [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html) connections for one powerful reason: the ability to reference [Security Groups (SGs)](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) across different VPCs. This functionality is incredibly useful for creating dynamic security rules without having to manage and update lists of CIDR ranges. However, I just realized that, thanks to an update I had missed, this is no longer necessary.

AWS introduced [**Security Group referencing support for AWS Transit Gateway**](https://aws.amazon.com/blogs/networking-and-content-delivery/introducing-security-group-referencing-for-aws-transit-gateway/), eliminating the need to use VPC Peering for this purpose and allowing for a complete migration to a centralized network architecture with Transit Gateway.

## What It Is and Why It Matters

This feature allows you to create an inbound rule in a Security Group (e.g., `sg-app` in `VPC-A`) that sources from a Security Group in another VPC (e.g., `sg-web` in `VPC-B`), as long as both VPCs are connected to the same Transit Gateway.

Previously, if you wanted `sg-app` to allow traffic from the `sg-web` instances, you had to use the private CIDR ranges of the subnet where the web instances were located, which is fragile and hard to maintain. Now, you can simply reference the SG's ID, and AWS handles the connectivity for you.

## How to Enable It

The feature is not fully enabled by default. For it to work, **it must be enabled in two places**:

1. **At the Transit Gateway level**: It is **disabled** by default. You must enable it explicitly.
2. **At the VPC Attachment level**: It is **enabled** by default.

If you don't enable the option on the Transit Gateway itself, it won't work, even if it's active on the attachments.

You can easily enable it on an existing Transit Gateway with the following AWS CLI command:

```shell
aws ec2 modify-transit-gateway \
  --transit-gateway-id <your-tgw-id> \
  --options SecurityGroupReferencingSupport=enable
```

## Conclusion

For me, this is a key change. It means I can simplify my network topology, remove the VPC Peering connections I was only maintaining for this reason, and centralize all connectivity through Transit Gateway without losing this important security feature.

For more details, you can read the official announcement on the [AWS Blog](https://aws.amazon.com/blogs/networking-and-content-delivery/introducing-security-group-referencing-for-aws-transit-gateway/).
