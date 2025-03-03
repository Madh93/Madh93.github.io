---
title: Direct JSON Field Access with CloudWatch Logs Insights
date: 2025-02-17
categories: [AWS, CloudWatch, Logs]
---

Did you know [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) can automatically parse JSON in the `@message` field? It means you can directly access the properties of the JSON object without using dot notation! This makes writing queries much cleaner and more intuitive.

Let's say your logs contain messages where `@message` is a JSON string like this: 

```json
{
  "level": "error",
  "status": 500,
  "error": "Internal Server Error",
  "other_data": "some_value"
}
```

And you want to filter for {{< sidenote "errors" >}}Notice that `level` works directly in the filter clause, even though it's a key inside the JSON stored in `@message`. You don't need to use `@message.level`.{{< /sidenote >}}. A query like this will work:

```shell
fields @timestamp, level, status, error, @logStream
| sort @timestamp desc
| filter level like 'error'
| limit 1000
```

The expected output would be:

| @timestamp               | level | status | error                 | @logStream      |
|--------------------------|-------|--------|-----------------------|-----------------|
| 2024-03-18T12:03:00.000Z | error | 503    | Service Unavailable   | your-log-stream |
| 2024-03-18T12:01:00.000Z | error | 500    | Internal Server Error | your-log-stream |
| 2024-03-18T11:58:00.000Z | error | 404    | Resource not found    | your-log-stream |
| 2024-03-18T11:55:00.000Z | error | 502    | Bad Gateway           | your-log-stream |
