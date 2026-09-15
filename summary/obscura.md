---
url: https://github.com/h4ckf0r0day/obscura
title: "Obscura"
author: h4ckf0r0day (SGavrl)
date_fetched: 2026-09-15
topics:
  - coding-agents-and-frameworks
  - mcp-and-tool-protocols
---

# Obscura (repo summary)

Obscura is a headless browser engine written from scratch in Rust for web scraping and AI-agent automation. It embeds real V8 (via `deno_core`) over a real DOM (`html5ever`), speaks the Chrome DevTools Protocol, and acts as a drop-in replacement for headless Chrome with Puppeteer and Playwright — while claiming ~30 MB memory vs Chrome's 200+, a ~70 MiB binary, and 51–85 ms page loads. Rendering is a first-class optional feature: Obscura implements its own CSS layout (vendored Taffy), text shaping (vendored cosmic-text), and CPU paint (tiny-skia + ab_glyph), so screenshots, live screencasting, and PDF export work without ever launching Chromium.

A separate `stealth` build adds anti-detection: a wreq/BoringSSL transport that presents a *consistent* Chrome 145 TLS ClientHello, ALPN and cipher order; per-session fingerprint randomization (GPU, screen, canvas, audio, battery); `navigator.webdriver` masking; native-function masking; and a 3,520-domain tracker blocklist. Security is default-on in the opposite direction: fetches to loopback, RFC1918, link-local (including the 169.254.169.254 cloud-metadata endpoint) and IANA special-purpose ranges are denied at both literal-host and DNS-resolution time to defeat rebinding, with `--allow-private-network` as the explicit dev opt-out.

Agent-facing surfaces are layered: a CLI (`fetch`, `scrape` with parallel `obscura-worker` processes over an NDJSON protocol, `serve` for CDP, `mcp`), a CDP WebSocket server (Target/Page/Runtime/DOM/Network/Fetch interception/IO streaming plus a custom `LP.getMarkdown` domain), and an MCP server (stdio or HTTP) exposing 38 `browser_*` tools — more than the README's table of 14 — including schema-driven `browser_extract`, form detection/filling, tab management, and markdown extraction designed for token-dense snapshots. Robustness is engineered for unattended fleets: one V8 isolate per connection serialized behind a `v8_lock`, a shared watchdog thread that terminates runaway JS (a budget `tokio::time::timeout` cannot preempt), script-deadline and per-module budgets, and a catch_unwind "anti-panic protocol" so a DOM-op panic degrades to `null` instead of aborting through V8's FFI frame.

Apache-2.0, single-author, very active (last commit 2026-09-14, PR #989); the README claims Cloudflare's Kitesurf agent-first browser began by porting Obscura to Workers. Unusually, the repo ships its own `AGENTS.md` and a bundled `skills/obscura/SKILL.md` — it is documented and maintained for agent operators first. The honest caveat: the independent CSS engine is the bet and the risk — long-tail CSS, media playback, and font rasterization may diverge from Chromium, and the headline benchmarks live in the author's own companion repo.
