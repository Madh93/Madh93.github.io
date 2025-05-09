---
title: Hidden CloudFront Invalidation Rate Limits
date: 2025-05-09
summary: CloudFront has an undocumented burst limit on invalidation requests that can cause "Rate exceeded" errors even under normal usage.
categories: [AWS, CloudFront, Invalidation, Rate Limits]
---

{{< admonition type="tip" title="TL;DR" >}}
[CloudFront](https://aws.amazon.com/cloudfront) has an **undocumented burst limit** on invalidation requests that can cause "Rate exceeded" errors even under normal usage.
{{< /admonition >}}

If you’ve ever automated cache clears with [CloudFront invalidations](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html), you might see intermittent:

```text
Error: Rate exceeded
```

This can happen due to an undocumented **burst rate limit** in [CloudFront's CreateInvalidation API](https://docs.aws.amazon.com/cloudfront/latest/APIReference/API_CreateInvalidation.html). The exact limit isn’t documented, but [some developers](https://github.com/aws/aws-sdk-js/issues/3983#issuecomment-1489381868) have found that sending several requests in the same second—maybe around 10—can cause this error. Since the real limit is unknown, it’s safest to slow down your requests and add throttling.

## Solutions

By implementing one (or more) of [these strategies](https://github.com/aws/aws-sdk-js/issues/3983#issuecomment-1617270477), you’ll avoid the undocumented CloudFront burst‐rate limit and keep your cache invalidations smooth and error‐free.

### 1. Throttle your Invalidation requests

Add a simple rate‐limiter so you never send more than 1 request per second (or up to 5/sec to stay safely under the burst). This ensures CloudFront won’t reject your calls even if your overall hourly count is low.

### 2. Batch more paths per call

Instead of calling `CreateInvalidation` separately for each path, submit a single call with multiple paths:

```js
const invalidation = await cloudfront.createInvalidation({
  DistributionId: 'YOUR_DIST_ID',
  InvalidationBatch: {
    Paths: {
      Quantity: paths.length,
      Items: paths
    },
    CallerReference: `${Date.now()}`
  }
}).promise();
```

Fewer API calls == less chance of hitting the burst limit.

### 3. Use CacheDisabled Cache Policy

If you need to frequently bypass the cache (for example, on every deploy), consider applying a [CacheDisabled](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-cache-policies.html#managed-cache-policy-caching-disabled) cache policy on specific behaviors instead of invalidating. This tells CloudFront to forward all requests to the origin and ignore its cache, eliminating the need for invalidations altogether.
