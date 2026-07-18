---
url: https://simonwillison.net/2026/Jul/16/grok-mermaid/
title: "Tool: Mermaid to Unicode box art (grok-mermaid)"
author: Simon Willison
date_fetched: 2026-07-18
date_published: 2026-07-16
site: simonwillison.net
tags: [tools, rust, webassembly, mermaid, grok, xai]
---

# Tool: Mermaid to Unicode box art (grok-mermaid)

[grok-mermaid](https://tools.simonwillison.net/grok-mermaid) is a new tool that I built that converts Mermaid diagram syntax into terminal-style Unicode box-drawing art, rendered entirely in your browser using WebAssembly.

It supports all of these Mermaid diagram types:

* flowcharts
* sequence
* state
* class
* ER diagrams

Other diagram types fall back to a framed source listing.

You can adjust the output width, copy out the diagram as text, or click "Copy link to this diagram" for a shareable link.

## How I built it

I learned about the underlying Rust renderer when I was exploring the codebase for the newly open-sourced Grok CLI coding agent. In [xai-org/grok-build](https://github.com/xai-org/grok-build) I found a file called `xai-grok-markdown/src/mermaid.rs` — a self-contained terminal renderer for Mermaid diagrams, written in Rust.

I figured it would be fun to try that out in a browser via WebAssembly, so I did.

The prompt I ran in Claude Code for web (Fable 5) is shared in [this pull request](https://github.com/simonw/tools/pull/40).

## Screenshot description

A screenshot of the tool shows a Mermaid diagram editor with source code on the left and the rendered flowchart on the right. All of the diagram layout is handled by the Rust code compiled to WebAssembly — there's no JavaScript reimplementation of any layout logic. The example chart illustrates the following flowchart:

Request received → Authenticated?
  ├─ yes → Rate limit OK?
  │         ├─ yes → Handle request ─┬─→ Audit log
  │         └─ no  → 429 Too Many Requests  │
  └─ no  → 401 Unauthorized          └─→ 200 OK

## How the WebAssembly build works

I compiled the Rust code (unmodified apart from two import lines) into a 163 KB WebAssembly module. There is no JavaScript reimplementation of any layout logic — everything runs the original Rust code via WASM. The build script and source are in the [grok-mermaid directory of my tools repository](https://github.com/simonw/tools/tree/main/grok-mermaid).

The module is a copyright 2023-2026 SpaceXAI and made available under the Apache License 2.0.

Posted 16th July 2026 at 12:33 am
