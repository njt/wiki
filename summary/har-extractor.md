---
url: https://github.com/azu/har-extractor
title: "har-extractor"
author: azu
date_fetched: 2026-08-01
date_published: 2018-06-20
topics:
  - developer-tools
---

A small CLI tool and library (MIT, by azu) that extracts HTTP response bodies from
HAR (HTTP Archive) files into a directory tree mirroring the original site structure.

The core is a single-pass synchronous pipeline: iterate every request/response entry,
decode the response body (handling base64 encoding), derive a filesystem path from
the request URL, and write the file. The entire library is 63 lines of TypeScript;
the CLI wrapper adds another 60 lines of JavaScript.

The URL-to-path conversion is the clever part. `humanize-url` strips protocols and
`www.` prefixes; `filenamify` sanitizes each segment for filesystem safety; and an
opinionated heuristic appends `/index.html` when the MIME type is `text/html` but
the URL path lacks `.html` — so a URL like `/wiki/.har` that returns HTML becomes
a filesystem path a browser can open directly.

The tool is deliberately simple. It loads the entire HAR into memory, writes
everything synchronously, has no per-entry error recovery, and does no caching or
content-encoding decompression. For the target use case — browser DevTools HAR
captures from a single page load, typically a few MB — none of these are problems.
For multi-GB HAR files from long recording sessions, they would be.
