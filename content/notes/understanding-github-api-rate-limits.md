---
title: Understanding GitHub API Rate Limits
date: 2025-03-06
categories: [GitHub, API, Rate Limits]
---

{{< admonition type="tip" title="TL;DR" >}}
[GitHub API](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api?apiVersion=2022-11-28) limits unauthenticated requests to **60 per hour per IP**. By using a personal access token, you can increase this limit to **5,000 requests per hour**.
{{< /admonition >}}

[GitHub](https://github.com/) applies [API rate limits](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api?apiVersion=2022-11-28) to ensure fair usage and system stability. These limits depend on the type of authentication used:

- **Unauthenticated Requests:** Limited to `60` requests per hour per IP address. This is often insufficient for applications making frequent API calls.
- **Authenticated Requests with Personal Access Tokens:** Using a personal access token boosts the limit to `5,000` requests per hour. This is ideal for developers working on projects that require continuous API interaction.
- **GitHub Apps or OAuth Apps:** These can achieve higher limits, especially when linked to [GitHub Enterprise Cloud](https://docs.github.com/en/enterprise-cloud@latest/admin/overview/about-github-enterprise-cloud) accounts. The rate limit can go up to `15,000` requests per hour under certain conditions.

## Checking current Rate Limit

To check your current rate limit status, you can use the [GitHub API](https://docs.github.com/en/rest/rate-limit/rate-limit?apiVersion=2022-11-28). Here's an example using `curl` to fetch the rate limit status:

```shell
curl -L \
-H "Authorization: Bearer <PERSONAL_ACCESS_TOKEN>" \
https://api.github.com/rate_limit
```

This request will return a JSON response showing your current rate limit status for different resources. For instance:

```json
{
  "resources": {
    "core": {
      "limit": 5000,
      "used": 1,
      "remaining": 4999,
      "reset": 1691591363
    },
    "search": {
      "limit": 30,
      "used": 12,
      "remaining": 18,
      "reset": 1691591091
    },
  },
  "rate": {
    "limit": 5000,
    "used": 1,
    "remaining": 4999,
    "reset": 1372700873
  }
}
```

This response indicates the maximum number of requests (`limit`), the number of requests you've made (`used`), the remaining requests (`remaining`), and when the rate limit resets (`reset`) in Unix timestamp.
