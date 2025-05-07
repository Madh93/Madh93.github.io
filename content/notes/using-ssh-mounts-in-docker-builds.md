---
title: Using SSH Mounts in Docker Builds
date: 2025-05-07
summary: Docker SSH mounts allow you to securely use your local SSH agent during docker build, for example to install private Git dependencies via SSH, without baking any keys into your image.
categories: [Docker, SSH, Secrets, BuildKit]
---

{{< admonition type="tip" title="TL;DR" >}}
[Docker SSH mounts](https://docs.docker.com/build/building/secrets/#ssh-mounts) allow you to securely use your local SSH agent during `docker build`, for example to install private Git dependencies via SSH, without baking any keys into your image.
{{< /admonition >}}

If you've ever needed to install a private Git dependency during a Docker build, such as a package in your `package.json` like:

```json
"dependencies": {
  "my-private-module": "git+ssh://git@github.com/your-org/my-private-module.git"
}
```

You might have run into authentication issues or resorted to insecure workarounds like copying SSH keys into the image.

Instead, [Docker SSH mounts](https://docs.docker.com/build/building/secrets/#ssh-mounts) offer a secure and temporary way to use your existing local SSH agent during image builds. This means your private SSH keys stay on your host machine and are never stored in the image.

## Enabling SSH mounts in a Dockerfile

In your Dockerfile, you might write:

```Dockerfile
FROM node:22

WORKDIR /app

COPY package.json package-lock.json ./

# Use your local SSH credentials to install private dependencies
RUN --mount=type=ssh npm ci

COPY . .
CMD ["node", "index.js"]
```

## Building the Image with SSH access

To build this image and allow the SSH mount, use the following command:

```shell
docker build --ssh default -t your-image-name .
```

This will give the container access to your local SSH agent **only during build time**, ensuring your credentials remain secure and are not embedded in the final image.

{{< admonition type="note" >}}
Make sure [Docker BuildKit](https://docs.docker.com/build/buildkit/) is **enabled**, as SSH mounts are only supported when BuildKit is active.
{{< /admonition >}}
