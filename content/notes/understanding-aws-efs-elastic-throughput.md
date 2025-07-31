---
title: Understanding AWS EFS Elastic Throughput
date: 2025-07-31
summary: AWS EFS Elastic Throughput is the recommended, pay-per-use throughput mode that automatically scales performance for spiky or unpredictable workloads, solving the limitations of the older Bursting and Provisioned modes.
categories: [AWS, EFS, Storage, Cost Optimization]
---

{{< admonition type="tip" title="TL;DR" >}}
AWS EFS Elastic Throughput is the recommended, pay-per-use throughput mode that automatically scales performance for spiky or unpredictable workloads, solving the limitations of the older Bursting and Provisioned modes.
{{< /admonition >}}

[Amazon EFS](https://aws.amazon.com/efs/) has historically offered two main [throughput modes](https://docs.aws.amazon.com/efs/latest/ug/performance.html#throughput-modes): **Bursting** and **Provisioned**. Bursting throughput scaled with the amount of data stored, which was not ideal for workloads with low storage but high, unpredictable access patterns. Provisioned throughput offered guaranteed performance but could be costly if you weren't consistently using the capacity you paid for.

To solve this, AWS introduced **EFS Elastic Throughput** in [late 2022](https://aws.amazon.com/blogs/aws/new-announcing-amazon-efs-elastic-throughput/), which is now the recommended default for most use cases.

## What is EFS Elastic Throughput?

EFS Elastic Throughput is a pay-per-use model where EFS automatically scales the throughput performance of your file system up or down based on your workload's activity.

Key advantages include:
- **No need to provision capacity**: You don't have to forecast your performance needs.
- **Cost-effective for spiky workloads**: You only pay for the data read and written, not for idle provisioned capacity.
- **Solves the low-storage problem**: Performance is no longer tied to the amount of data in your file system, making it perfect for applications with small datasets but high throughput requirements.

Because of this flexibility and cost model, AWS now recommends EFS Elastic Throughput for any application with spiky, unpredictable, or hard-to-forecast performance needs.

{{< admonition type="note" title="A Note on Switching Modes" >}}
It's important to be aware of a restriction when changing your file system's throughput mode. After you switch to **Provisioned throughput** or modify the provisioned amount, you are locked in for **24 hours**. During this period, you **cannot**: switch back to `Elastic`/`Bursting` mode or decrease the `Provisioned` throughput amount.
{{< /admonition >}}

For more details, you can read the official announcement on the [AWS Blog](https://aws.amazon.com/blogs/aws/new-announcing-amazon-efs-elastic-throughput/).
