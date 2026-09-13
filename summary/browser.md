---
url: https://github.com/lightpanda-io/browser
title: "Lightpanda Browser"
author: Lightpanda (Selecy SAS) — Francis Bouvier, Pierre Tachoire
date_fetched: 2026-09-13
topics:
  - coding-agents-and-frameworks
  - mcp-and-tool-protocols
---

# Lightpanda Browser (repo summary)

A headless browser written from scratch in Zig (~227K lines of Zig, ~4K of Rust; AGPL-3.0), not a Chromium fork or WebKit patch. It embeds v8 for JavaScript, html5ever (via a Rust FFI workspace) for parsing, and libcurl for HTTP — but has no layout or rendering engine at all. The pitch is performance where it matters for automation: the README benchmarks ~123MB peak and ~5s for 100 pages vs ~2GB and ~46s for headless Chrome.

What makes it more than a fast scraper is that it is an agent runtime first. A single tool layer (`src/browser/tools.zig`, a closed enum of 28 text tools: `tree`, `markdown`, `extract`, `goto`, `click`, `fill`, `search`, …) backs every consumption surface: an in-process LLM agent (`lightpanda agent`), a native MCP server (stdio, or HTTP with per-connection browsing sessions routed by `Mcp-Session-Id`), CDP and WebDriver BiDi servers for Puppeteer/Playwright, and a custom CDP `lp.` domain. `/save` compiles a finished agent session into PandaScript — vanilla JS with native browser primitives — that replays deterministically and token-free via `lightpanda run`.

Agent-side engineering is deliberate: a shared `driver_guidance` system prompt (same string for the agent and MCP `initialize` instructions), a cheap-to-expensive read ladder (semantic `tree` → `nodeDetails` → `markdown` → `html`), schema-driven `extract`, CSS-selector-only interactions so sessions stay replayable, an anti-confabulation synthesis turn, `$LP_*` placeholders that keep secrets out of model context, and network guardrails (comptime CIDR SSRF filter, robots gate, GCRA rate limiter, Web Bot Auth signing). It also implements the WebMCP draft: pages can register tools through a `ModelContext` JS API that agents invoke over CDP.
