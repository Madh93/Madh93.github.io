---
title: Get YouTube Subscriptions RSS Feed from an OPML File
date: 2025-06-16
summary: You can bypass the YouTube algorithm and get updates from all your subscribed channels by exporting them as an OPML file using YouTube-Subscriptions-RSS to be imported in your RSS reader.
categories: [YouTube, RSS, OPML, Automation]
---

{{< admonition type="tip" title="TL;DR" >}}
You can bypass the [YouTube](https://www.youtube.com/) algorithm and get updates from all your subscribed channels by exporting them as an OPML file using [YouTube-Subscriptions-RSS](https://github.com/jeb5/YouTube-Subscriptions-RSS) to be imported in your RSS reader.
{{< /admonition >}}

I've found that my [YouTube](https://www.youtube.com/) homepage is often filled with recommended videos rather than the latest uploads from channels I'm actually subscribed to. This makes it easy to miss out on new content from creators I want to follow.

Today I learned a simple way to solve this: using [RSS feeds](https://en.wikipedia.org/wiki/RSS). While YouTube no longer provides a native feature to export subscriptions, you can use a community-built tool to generate an `OPML` file, which is a standard format for sharing subscription lists between RSS readers.

## How to Generate the OPML File

We will use the script provided by [YouTube-Subscriptions-RSS](https://github.com/jeb5/YouTube-Subscriptions-RSS). The process involves running a small piece of JavaScript on your subscriptions page.

1. Navigate to your [YouTube subscriptions page](https://youtube.com/feed/channels). You must be logged in.
2. Open the JavaScript console in your browser.
3. Copy the JavaScript code from the [YouTube-Subscriptions-RSS](https://github.com/jeb5/YouTube-Subscriptions-RSS?tab=readme-ov-file#script) repository and paste it into the console, then press Enter. Alternatively, you can create a [bookmarklet](https://github.com/jeb5/YouTube-Subscriptions-RSS?tab=readme-ov-file#bookmarklet) for easier reuse.
4. The script will run and automatically trigger the download of an OPML file.

## Import into Your RSS Reader

The downloaded OPML file contains the name and the RSS feed URL for every channel you are subscribed to. You can directly import this file into most modern RSS readers (I like [Miniflux](https://miniflux.app/)). The reader will automatically parse the file and add all the channels to your feed list.

This way, I can get chronological updates from all my subscriptions in one place, putting me back in control of my own feed.
