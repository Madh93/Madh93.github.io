---
title: Reading Records from an AWS Kinesis Data Stream
date: 2025-06-24
summary: Use a combination of AWS CLI commands like get-shard-iterator and get-records to fetch raw, Base64-encoded data from a Kinesis Data Stream for debugging.
categories: [AWS, Kinesis, CLI, Debugging]
---

{{< admonition type="tip" title="TL;DR" >}}
Use a combination of AWS CLI commands like `get-shard-iterator` and `get-records` to fetch raw, Base64-encoded data from a Kinesis Data Stream for debugging.
{{< /admonition >}}

When you need to debug the content of an [Amazon Kinesis Data Stream](https://aws.amazon.com/kinesis/data-streams/), for example, to inspect the fields being sent by [CloudFront Real-time logs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/real-time-logs.html), you need a way to fetch and read the raw records directly from the stream.

Reading from a Kinesis stream is not as simple as tailing a log file, because you must first target a specific shard within the stream and get a temporary pointer (a shard iterator) to the position from which you want to start reading.

### The Process

1. **describe-stream**: Get the ID of a shard you want to inspect.
2. **get-shard-iterator**: Obtain an iterator for that shard. You can specify where to start reading from, for instance, `TRIM_HORIZON` for the oldest record or `LATEST` for new ones.
3. **get-records**: Use the iterator to fetch the records.
4. **Decode**: The record's data is Base64 encoded, so it must be decoded to be human-readable.

### All-in-One Command

You can combine these steps into a single, powerful one-liner to quickly get the first record from the first shard of a stream. This is incredibly useful for a quick debugging check.

```shell
aws kinesis get-records \
  --shard-iterator $(aws kinesis get-shard-iterator --stream-name $KINESIS_STREAM_NAME --shard-iterator-type 'TRIM_HORIZON' --shard-id $(aws kinesis describe-stream --stream-name $KINESIS_STREAM_NAME --query 'StreamDescription.Shards[0].ShardId' | tr -d '"') --output text) \
  --query 'Records[0].Data' \
  --output text | base64 -d | sed -e 'y/\t/\n/'
```

Let's break it down:
- The inner-most command `aws kinesis describe-stream ...` finds the ID of the very first shard.
- This ID is passed to `aws kinesis get-shard-iterator ...` to get an iterator starting from the beginning of the stream (`TRIM_HORIZON`).
- The main `aws kinesis get-records ...` command uses this iterator to fetch records and extracts (`--query`) the `Data` field of the first record.
- Finally, the output is piped to `base64 -d` for decoding and then to `sed -e 'y/\t/\n/'` to replace the tab characters (common in CloudFront logs) with newlines, making the output easy to read.
