---
title: Parsing Legacy CloudFront Standard Logs to JSON
date: 2025-03-07
categories: [AWS, CloudFront, Logs, JSON, Python]
---

{{< admonition type="tip" title="TL;DR" >}}
[Legacy CloudFront Standard Logs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/standard-logging-legacy-s3.html) (a.k.a access logs) are based on [W3C Extended Log File format](https://www.w3.org/TR/WD-logfile.html). These logs can be parsed into JSON format using a Python script.
{{< /admonition >}}

[CloudFront](https://aws.amazon.com/cloudfront/) allows you to enable logging to capture detailed information about every request it receives. These logs, known as **standard logs** or **access logs**, help you analyze traffic patterns and troubleshoot issues.

CloudFront offers two versions of [standard logging](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/AccessLogs.html):

- **Legacy:** Sends access logs exclusively to Amazon S3.
- **V2:** Provides more flexibility by supporting multiple delivery destinations. Depending on where you send the logs, you can choose from different formats: JSON, Plain, W3C, Raw, or Parquet.

## Legacy CloudFront Standard Logs use W3C Format

Legacy logs are stored in [W3C Extended Log Format](https://www.w3.org/TR/WD-logfile.html), a structured text format where each entry consists of space-separated values. The first lines of the file contain metadata (`#Fields:`) specifying which fields appear in each log entry.

While this format is widely supported, parsing it efficiently requires some preprocessing. That’s why I ended up creating a script to convert them to JSON 😅

## Parsing W3C Log Format

To convert these the CloudFront logs in W3C format file to JSON, you can use the following Python script:

```python
import json
import sys

fields = []
logs = []

for line in sys.stdin:
    # Extract fields
    if line.startswith("#Fields:"):
        fields = [field.strip() for field in line.strip().replace("#Fields: ", "").split()]
        continue
    # Ignore other comments
    if line.startswith("#"):
        continue
    # Extract values
    values = line.strip().split('\t')
    # Add a dictionary with the fields and values
    logs.append(dict(zip(fields, values)))

# Write the list of logs as JSON to stdout
json.dump(logs, sys.stdout, indent=4)
```

To use this script, you can pipe the log data from S3 through `gzip` to decompress it, and then through the Python script to convert it to JSON. Here's how you can do it:

```shell
aws s3 cp 's3://my-cf-logs/E44BQA1CC1GO6W.2025-03-07-16.da452xa1.gz' - | gzip -d | python parser.py
```

The JSON output will look something like this:

```json
[
    {
        "date": "2025-03-07",
        "time": "16:06:40",
        "x-edge-location": "MAD53-P3",
        "sc-bytes": "618",
        "c-ip": "23.195.122.99",
        "cs-method": "GET",
        "cs(Host)": "1y2yasjsd8.cloudfront.net",
        "cs-uri-stem": "/childrenContextMenu",
        "sc-status": "200",
        "cs(Referer)": "https://foo.bar.com/",
        "cs(User-Agent)": "Mozilla/5.0%20(X11;%20Linux%20x86_64)%20AppleWebKit/537.36%20(KHTML,%20like%20Gecko)%20Chrome/133.0.0.0%20Safari/537.36",
        "cs-uri-query": "-",
        "cs(Cookie)": "-",
        "x-edge-result-type": "Miss",
        "x-edge-request-id": "OA6u0S41z4UFD1xXph6TJFCy8kQkcHK8vMmrZUBd4uaorkwueZw2SA==",
        "x-host-header": "foo.bar.com",
        "cs-protocol": "https",
        "cs-bytes": "46",
        "time-taken": "0.078",
        "x-forwarded-for": "-",
        "ssl-protocol": "TLSv1.3",
        "ssl-cipher": "TLS_AES_128_GCM_SHA256",
        "x-edge-response-result-type": "Miss",
        "cs-protocol-version": "HTTP/2.0",
        "fle-status": "-",
        "fle-encrypted-fields": "-",
        "c-port": "36314",
        "time-to-first-byte": "0.078",
        "x-edge-detailed-result-type": "Miss",
        "sc-content-type": "application/json;charset=utf-8",
        "sc-content-len": "188",
        "sc-range-start": "-",
        "sc-range-end": "-"
    }
]
```
