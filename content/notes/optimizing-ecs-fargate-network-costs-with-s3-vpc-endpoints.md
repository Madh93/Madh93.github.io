---
title: Optimizing ECS Fargate Network Costs with S3 VPC Endpoints
date: 2025-02-27
summary: Use S3 Gateway VPC Endpoints to reduce network costs when pulling private ECR images in ECS Fargate tasks deployed in private subnets.
categories: [AWS, ECS, Fargate, VPC, Cost Optimization]
---

{{< admonition type="tip" title="TL;DR" >}}
Use [S3 Gateway VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html) to reduce network costs when pulling **private ECR images** in ECS Fargate tasks deployed in **private subnets**.
{{< /admonition >}}

When running [ECS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html) tasks in **private subnets**, you often need to pull container images from a repository like [Amazon Elastic Container Registry (ECR)](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html). This can lead to significant network costs as the traffic is routed through a NAT Gateway. A handy solution to this is using VPC Endpoints.

Now, keep in mind that only [Gateway-type VPC Endpoints (like S3 and DynamoDB)](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html) are **free**. Interface Endpoints will still incur charges.

## The Optimization

Here's the {{< sidenote "trick" >}}While this optimization can save you money, remember the saying: "Premature optimization is the root of all evil."  Measure your network traffic and costs before implementing this solution to ensure it is worthwhile.{{< /sidenote >}}: When you pull a **private** image from ECR, the actual image data is stored in S3. By creating an [S3 Gateway VPC Endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html), you allow your Fargate tasks to access those images directly via the AWS internal network, bypassing the NAT Gateway and saving you bandwidth costs.

{{< admonition >}}
Public images hosted in [ECR Public Gallery](https://gallery.ecr.aws/), or other public repositories, will still need to go through the NAT Gateway (or other internet access solutions) to be pulled. This is because the S3 Endpoint will only route traffic to S3 buckets *within your account* (or those to which you've granted access).
{{< /admonition >}}

If you're interested in further optimizing network costs for ECS, check out this article: [Cost Optimisation on AWS: Navigating NAT Charges with Private ECS Tasks on Fargate](https://dev.to/chayanikaa/cost-optimisation-on-aws-navigating-nat-charges-with-private-ecs-tasks-on-fargate-21lp)
