---
url: https://tools.simonwillison.net/llm-cliche-highlighter
title: LLM Cliché Highlighter
author: Simon Willison
date_fetched: 2026-07-21
date_published: unknown
---

# LLM Cliché Highlighter

A browser-based tool by Simon Willison that scans text for phrases commonly overused by large language models. Paste text or load from a URL; highlights sentences matching known LLM clichés. Chain-style patterns like "no X, no Y" get a badge showing chain item count. Hovering over a highlight reveals which specific cliché triggered the match.

## Features

- **Real-time analysis**: runs as you type
- **Patterns view**: lists the cliché patterns being matched
- **Highlighted text panel**: displays flagged sentences with pattern match and chain item count
- **Matches section**: logs results
- **"Show just the highlights" toggle**: filters view to only flagged sentences
- **Load URL**: fetches and analyzes text from an external URL
- **Load example** and **Clear** buttons

## Technical Implementation

- Single self-contained HTML file
- Self-tests embedded between marker comments (`// ==== impl start ====` / `// ==== impl end ====` and `// ==== tests start ====` / `// ==== tests end ====`)
- Headless execution via Node.js one-liner that extracts implementation, evals it, runs self-tests, and exits with code 1 on failure
- Live at: https://tools.simonwillison.net/llm-cliche-highlighter

## Pattern Examples

- Chain patterns: "no X, no Y" (gets a badge counting chain items)
- Full pattern list is embedded in the JavaScript implementation (not visible in the page text)
