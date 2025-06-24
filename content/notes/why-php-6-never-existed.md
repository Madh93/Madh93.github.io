---
title: Why PHP 6 Never Officially Existed
date: 2025-06-21
summary: PHP skipped version 6 because the project, which was primarily focused on adding native Unicode support, stalled due to complexity and performance issues. To avoid confusion with the failed effort and its associated books, the community voted to name the next major release PHP 7.
categories: [PHP, Programming Languages, History]
---

{{< admonition type="tip" title="TL;DR" >}}
PHP skipped version 6 because the project, which was primarily focused on adding native Unicode support, stalled due to complexity and performance issues. To avoid confusion with the failed effort and its associated books, the community voted to name the next major release PHP 7.
{{< /admonition >}}

If you follow the history of [PHP](https://www.php.net/), you might notice a curious jump: the versioning goes directly from PHP 5.6 to PHP 7.0, completely skipping version 6. This wasn't an accident but a deliberate decision made by the community for several key reasons.

## The Ambitious Goal of PHP 6

The primary mission for what was to become PHP 6 was a massive undertaking: to add native, core-level support for [Unicode (UTF-8)](https://en.wikipedia.org/wiki/Unicode). This was seen as a crucial step to modernize the language and handle international character sets properly. Development on this began as early as 2005.

## A Troubled Development

The project, however, ran into significant difficulties. The implementation of Unicode support at the core of the [Zend Engine](https://en.wikipedia.org/wiki/Zend_Engine) was complex, caused performance degradation, and stalled the development of other features. While the PHP 6 project was struggling, many other valuable features were developed and simply backported to the PHP 5.x branch, resulting in popular releases like 5.3 and 5.4.

By 2010, it became clear that the PHP 6 project as originally conceived had failed and was officially abandoned.

## The Move to PHP 7

With the PHP 6 project abandoned, the PHP community needed a new major version to introduce significant improvements, most notably the new "PHP Next-Gen" (PHPNG) engine, which promised massive performance gains.

The name "PHP 6" was already tarnished by the failed project. Furthermore, numerous books and articles had already been published about the planned (but never released) PHP 6, which would cause confusion.

To signal a fresh start and avoid this confusion, the core developers held a vote. The community decided to officially skip the number 6 and name the next major release **PHP 7**. This new version, released in 2015, represented a true leap forward, focusing on performance and new language features, and leaving the troubled legacy of the PHP 6 Unicode project behind.
