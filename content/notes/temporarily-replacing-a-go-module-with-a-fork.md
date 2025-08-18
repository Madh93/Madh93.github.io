---
title: Temporarily Replacing a Go Module with a Fork
date: 2025-08-18
summary: Use 'go mod edit -replace' to point a Go dependency to your personal fork. This is perfect for unblocking development while you wait for an upstream pull request to be merged.
categories: [Go, Git, CLI, Development, Workflow]
---

{{< admonition type="tip" title="TL;DR" >}}
Use `go mod edit -replace` to point a Go dependency to your personal fork. This is perfect for unblocking development while you wait for an upstream pull request to be merged.
{{< /admonition >}}

I was recently contributing to the [terraform-provider-gitea](https://gitea.com/gitea/terraform-provider-gitea) project and needed new functionality in its dependency, the [gitea.com/gitea/go-sdk](https://gitea.com/gitea/go-sdk). I implemented the changes and opened a [pull request](https://gitea.com/gitea/go-sdk/pulls/722) in the SDK repository, but I didn't want my provider development to be blocked while waiting for the PR to be reviewed and merged.

## The Solution

The Go toolchain provides a [powerful command](https://go.dev/ref/mod#go-mod-edit) to handle this exact scenario. You can temporarily [replace a module dependency](https://go.dev/doc/modules/gomod-ref#replace) with a different source, such as a local path or, in this case, a specific branch of a personal fork.

To do this in a single line, I used the following command:

```shell
go mod edit -replace=gitea.com/gitea/go-sdk=$(go list -m gitea.com/Madh93/go-sdk@extend-push-mirrors | tr ' ' '@')
```

Let's break down how this works:

- **`go mod edit -replace`**: This is the main command to add a `replace` directive to the `go.mod` file.
- **`gitea.com/gitea/go-sdk`**: This is the original module path that we want to replace.
- **`go list -m gitea.com/Madh93/go-sdk@extend-push-mirrors`**: This command resolves the head of my `extend-push-mirrors` branch to a specific commit, outputting its module path and pseudo-version (e.g., `gitea.com/Madh93/go-sdk v0.0.0-20250806...`).
- **`| tr ' ' '@'`**: The output of `go list` has a space between the path and the version. This pipe command replaces that space with an `@`, creating the `path@version` format that `go mod edit` expects.

## The Result

After running the command, my `go.mod` file was updated with the following line:

```go
replace gitea.com/gitea/go-sdk => gitea.com/Madh93/go-sdk v0.0.0-20250806161731-d8c06da0eae3
```

This directive tells the Go toolchain: whenever any part of this project asks for `gitea.com/gitea/go-sdk`, use this specific commit from my fork instead.

This is an invaluable technique for maintaining development velocity. Just remember to remove the `replace` directive and update to the official version once your upstream pull request is merged!
