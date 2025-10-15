---
title: AWS Launches Instant EBS Volume Clones
date: 2025-10-15
summary: AWS has launched EBS Volume Clones, a new feature to create nearly instant copies of encrypted EBS volumes within the same Availability Zone. It's much faster than the old snapshot-and-restore method for creating dev/test environments.
categories: [AWS, EBS, Storage]
---

{{< admonition type="tip" title="TL;DR" >}}
AWS has launched EBS Volume Clones, a new feature to create nearly instant copies of encrypted EBS volumes within the same Availability Zone. It's much faster than the old snapshot-and-restore method for creating dev/test environments.
{{< /admonition >}}

Creating copies of production data for development or testing has always been a slow, multi-step process on AWS. The standard workflow involved taking an [**EBS snapshot**](https://aws.amazon.com/ebs/snapshots/), waiting for it to complete, and then creating a new volume from that snapshot. This process was cumbersome and introduced significant delays, especially waiting for the new volume to hydrate from S3 before it could deliver full performance.

## The Solution: Instant EBS Volume Clones

In a recent [**announcement blog post**](https://aws.amazon.com/blogs/aws/introducing-amazon-ebs-volume-clones-create-instant-copies-of-your-ebs-volumes/), AWS introduced **EBS Volume Clones**. This new feature allows you to create a point-in-time copy of an EBS volume with a single API call or console click. The new cloned volume is available and fully usable within seconds, as the data is copied lazily in the background without impacting the performance of the source volume. This drastically speeds up workflows like refreshing a staging database or setting up isolated test environments.

## Key Features and Considerations

EBS Volume Clones are a powerful tool, but it's important to understand how they work and their limitations:

* **Use Case, Not a Backup:** Clones are designed for speed within the **same Availability Zone**. They are crash-consistent and complement, but do **not** replace, EBS snapshots. Snapshots remain the recommended solution for backup, disaster recovery, and cross-AZ durability.
* **Key Limitations:** At launch, you can only create clones from **encrypted** source volumes. You can only create one copy at a time from a source volume, and tags are not carried over to the clone.
* **Pricing Model:** The cost has two parts: a one-time fee for the cloning operation (charged only on the *written blocks* of the source volume, not its total provisioned size) and the standard ongoing EBS storage cost for the newly created volume.

For a full list of quotas and considerations, always check the [**official documentation**](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-copying-volume.html).
