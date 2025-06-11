---
title: Solving Amazon Data Firehose Bottlenecks
date: 2025-06-11
summary: When Amazon Data Firehose uses a Lambda for transformation, high data volume can exceed the AWS Lambda concurrent execution limit, causing processing delays. The most effective solution is to increase Firehose's buffer size to reduce the number of Lambda invocations.
categories: [AWS, Kinesis, Firehose, Lambda]
---

{{< admonition type="tip" title="TL;DR" >}}
When [Amazon Data Firehose](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html) uses a [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) for transformation, high data volume can exceed the AWS Lambda concurrent execution limit (default: [1000](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)), causing processing delays. The most effective solution is to **increase Firehose's buffer size** to reduce the number of Lambda invocations.
{{< /admonition >}}

Today I troubleshooted a data pipeline where logs from CloudFront were being sent to a [Kinesis Data Stream](https://docs.aws.amazon.com/streams/latest/dev/introduction.html), processed by an [Amazon Data Firehose](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html) with a Lambda for transformation, and finally sent to a logging destination. The system was experiencing significant delays and Firehose was reporting a high number of `Destination error` messages with the reason: `The Lambda concurrent execution limit is exceeded.`

The core issue was not a failure in the Lambda's code, but a bottleneck in the architecture's ability to scale.

## Understanding the Data Flow

The data pipeline is designed for high-throughput, real-time processing:

1. **Ingestion (Kinesis Data Stream):** Receives a massive, continuous flow of logs from the source.
2. **Consumption (Amazon Data Firehose):** Reads records from the data stream. To keep up, it reads in parallel and groups records into internal buffers.
3. **Transformation (AWS Lambda):** When a buffer is full (based on size in MB or time in seconds), Firehose invokes a Lambda function to process that batch of records.

With a high volume of incoming data, Firehose fills many buffers simultaneously. It attempts to invoke the transformation Lambda for each of these buffers in parallel. This can easily exceed the default account-wide concurrent Lambda execution limit of [1000](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html). When AWS throttles the invocations, Firehose fails to process the batch and schedules it for a retry, causing delays.

A key metric to monitor for this issue on the source Kinesis Data Stream is `GetRecords.IteratorAgeMilliseconds`. A rising value for the Firehose consumer indicates it is falling behind and not processing data as fast as it arrives.

## How to Fix the Bottleneck

The goal is to relieve the pressure on Lambda. Here are three approaches, presented in a balanced style.

1. **Increase the Firehose Buffer Size (Most Recommended):** This is the most effective and cost-efficient solution. It involves **increasing the buffer size and buffer interval** in your Firehose configuration. For example, changing from `1 MB` to `5 MB` and from `60` to `300` seconds. Doing this forces Firehose to create larger data batches, which drastically reduces how often it needs to invoke Lambda.
2. **Request a Lambda Concurrency Limit Increase:** If your data volume is so high that adjusting the buffer isn't enough, the next step is to **request an increase to the "Concurrent executions" quota** in the AWS Service Quotas console. Keep in mind that this is a "brute force" solution that can increase costs and may impact other applications in the same region by consuming shared resources.
3. **Optimize the Lambda Function:** Making the function run faster frees up concurrency slots sooner. A simple way to achieve this is to **increase the memory allocated to the Lambda function.** More memory also provides more vCPU power, which speeds up the processing of each batch and reduces the average duration of each invocation.
