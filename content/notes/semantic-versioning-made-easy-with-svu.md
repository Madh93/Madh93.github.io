---
title: Semantic Versioning Made Easy with svu
date: 2025-03-05
categories: [Versioning, Release, CI/CD, Go, Git]
---

{{< admonition type="tip" title="TL;DR" >}}
[svu](https://github.com/caarlos0/svu) is a lightweight CLI tool that generates semantic versions based on your Git log and follows [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). It is the perfect companion for [GoReleaser](https://goreleaser.com/).
{{< /admonition >}}

Automating semantic versioning can be cumbersome, especially when dealing with CI/CD pipelines. [svu](https://github.com/caarlos0/svu) simplifies this by:

- Automatically determining the next semantic version based on Git commits.
- Supporting [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) to derive version increments (major, minor, patch).
- Being lightweight, portable, and designed for seamless integration with Go projects and [GoReleaser](https://goreleaser.com/).

## Examples

### Get the next version

If your Git repository follows Conventional Commits, you can run:

```shell
svu next
```

This will output the next semantic version based on your commit history. For example, if your last version was `v1.2.3` and you made a commit with `feat: add new API`, this command will return `v1.3.0`.

### Bump a specific version

If you want to manually specify the next version type, you can do from an existing `v1.2.3` version:

```shell
svu patch # Outputs the next patch version, e.g., v1.2.4
svu minor # Outputs the next minor version, e.g., v1.3.0
svu major # Outputs the next major version, e.g., v2.0.0
```

## Using with GoReleaser

[GoReleaser does not create any tags](https://goreleaser.com/cookbooks/semantic-release/), it just runs on what is already there. You can integrate `svu` to automatically determine the version:

```shell
git tag "$(svu next)"
git push --tags
goreleaser release --clean # Or run this in CI/CD (e.g., GitHub Actions)
```

This ensures that every release follows semantic versioning without manual intervention.
