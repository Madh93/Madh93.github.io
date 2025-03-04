---
title: Rewriting Rust with Even More Rust
date: 2025-03-04
categories: [Rust, Compilers, Programming Languages]
---

{{< admonition type="tip" title="TL;DR" >}}
Rust's compiler is written in [Rust](https://www.rust-lang.org/), thanks to a process called [bootstrapping](https://en.wikipedia.org/wiki/Bootstrapping_(compilers)).
{{< /admonition >}}

I recently stumbled upon a [Reddit thread](https://www.reddit.com/r/rust/comments/1j13qos/what_language_is_rust_written_in_like_python_is/) where someone asked what language [Rust](https://www.rust-lang.org/) is written in. The answer? **Rust**. At first, I thought it was a joke, after all, many projects are being rewritten in Rust these days. But it turns out to be true!

Originally, Rust’s compiler was written in [OCaml](https://ocaml.org/), but once Rust became stable enough, its compiler was rewritten in Rust itself using a process called bootstrapping.

## What is Bootstrapping?

[Bootstrapping](https://en.wikipedia.org/wiki/Bootstrapping_(compilers)) is a technique in compiler development where a language's compiler is written in that same language. Since a compiler is just another program, it needs to be compiled itself before it can be used. Here’s how the process typically works:

1. **Initial Compiler in Another Language:** A new programming language needs a compiler, so its first version is written in an existing language (e.g., OCaml for Rust).
2. **Self-Hosting:** Once the language is mature enough, a new compiler is written using the language itself.
3. **Bootstrapping:** The old compiler compiles the new one, and from then on, the language is self-sufficient.

Imagine we invent a new language called `FooLang`. At first, we write a simple compiler in Python that translates `FooLang` code into machine code. Once `FooLang` has enough features, we write a new compiler in `FooLang` itself and use the original Python-based compiler to compile it. Now, `FooLang` can compile itself!

## Why Use Bootstrapping?

- **Optimization:** Writing the compiler in its own language allows better performance and integration.
- **{{< sidenote "Dogfooding" >}} Eating your own dog food or "dogfooding" is the practice of using one's own products or services.{{< /sidenote >}}:** Developers use the language to improve it continuously.
- **Portability:** Once bootstrapped, the language can be built on different platforms without relying on an external compiler.

If you're interested in learning more about bootstrapping, check out the next video: {{< youtube PjeE8Bc96HY >}}
