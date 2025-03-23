---
title: Preloading SSM Secrets in AWS Lambda with Crypteia
date: 2025-03-23
categories: [AWS, SSM, Lambda, Crypteia]
---

{{< admonition type="tip" title="TL;DR" >}}
[Crypteia](https://github.com/rails-lambda/crypteia) is an [AWS Lambda extension](https://docs.aws.amazon.com/lambda/latest/dg/lambda-extensions.html) for any runtime to preload SSM Parameters as secure environment variables at function startup.
{{< /admonition >}}

By specifying environment variables in your deployment (for example, setting `SECRET: x-crypteia-ssm:/myapp/SECRET`), Crypteia fetches the corresponding secret from [AWS SSM Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) and seamlessly replaces the placeholder with the actual value. This process occurs only once during the function's initialization, ensuring minimal runtime overhead.

## Container Image Integration

Below is a sample Dockerfile that demonstrates how to integrate Crypteia into your AWS Lambda container image:

```dockerfile
FROM public.ecr.aws/docker/library/python:3.13-slim
COPY --from=ghcr.io/rails-lambda/crypteia-extension-debian:2 /opt /opt
ENV LD_PRELOAD=/opt/lib/libcrypteia.so \
    PYTHONPATH=/opt/crypteia/python
```

{{< admonition type="note" >}}
If you are using Python you will need to set the `PYTHONPATH` environment variable with the Crypteia package in order for things to just work.
{{< /admonition >}}

This configuration may vary depending on your base image. For further details, please refer to the Crypteia [installation guide](https://github.com/rails-lambda/crypteia?tab=readme-ov-file#installation).

## Usage with Lambda Environment Variables

Crypteia allows you to manage SSM Parameter Store secrets in your Lambda environment variables in two ways.

### Direct Variable Replacement

You can define an environment variable with a direct SSM parameter. For example, if you set:

```yml
SECRET: x-crypteia-ssm:/myapp/SECRET
```

When your Lambda function initializes, any call to `getenv("SECRET")` (or the equivalent in your runtime) will return the secret value (e.g., `1A2B3C4D5E6F`) fetched from {{< sidenote "SSM Parameter Store" >}} Don't forget to update the required IAM permissions.{{< /sidenote >}}.

### Grouped Parameter fetch using a Path

Alternatively, you can configure Crypteia to fetch multiple parameters by specifying a group prefix. For instance, consider the following setup:

```yml
X_CRYPTEIA_SSM: x-crypteia-ssm-path:/myapp/env
DB_URL: x-crypteia
NR_KEY: x-crypteia
```

Here, the environment variable `X_CRYPTEIA_SSM` tells Crypteia to retrieve all parameters under the `/myapp/env` path. Then, for each subsequent environment variable (like `DB_URL` and `NR_KEY`) that uses the `x-crypteia` placeholder, Crypteia replaces it with the corresponding value from the SSM path.
