---
url: https://tools.simonwillison.net/llm-cliche-highlighter
title: "LLM Cliché Highlighter"
author: Simon Willison
date_fetched: 2026-07-21
topics:
  - developer-tools
---

A browser-based tool that scans text for phrases commonly overused by large language models. Paste text or load from a URL; sentences matching known LLM clichés are highlighted inline. Hovering over a highlight reveals which specific cliché triggered the match.

Chain-style patterns like "no X, no Y" get a badge showing the chain item count. A "Show just the highlights" toggle filters the view to only flagged sentences, making it easy to spot AI-generated prose at a glance.

The tool is a single self-contained HTML file with embedded JavaScript. Self-tests live between marker comments in the source, enabling headless validation via a one-liner Node.js invocation that extracts the implementation, runs the tests, and exits with a failure code if any break.

The cliché pattern list is embedded in the JavaScript implementation and visible in the tool's Patterns view.
