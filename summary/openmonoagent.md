---
url: https://www.openmonoagent.ai/
title: OpenMonoAgent — Terminal-Native Local Coding Agent
author: StartupHakk LLC
date_fetched: 2026-07-05
date_published: 2025
---

# OpenMonoAgent.ai

OpenMonoAgent is a terminal-native coding agent powered by local LLMs. It runs entirely on the user's own hardware, requires no API keys, and offers unlimited tokens at no cost. The project is in beta, built by StartupHakk LLC.

Founding quote: "AI shouldn't be a subscription you rent. It should be *infrastructure you own*"

## Architecture

- Built on C#/.NET — "AI tooling should be infrastructure, not a subscription"
- Embedded llama.cpp — two hardware paths: NVIDIA GPU runs Qwen3.6 27B; CPU fallback runs Qwen3.6 35B A3B
- Docker-native sandboxing — agent mounts project into container, "can't leave the box"
- Fully offline — no cloud dependency, no data leakage
- Outbound-only dual-box mode for separating laptop client from home GPU rig

## Features

1. Embedded inference, zero setup — hardware auto-detection, no config or API keys
2. TUI for long sessions — full-screen ANSI with streaming tokens, tok/s meter, context-window tracking, auto-compaction
3. Docker-sandboxed — permission gates on destructive operations
4. 20 tools + MCP — file I/O, shell, search, web fetch, LSP, patches, sub-agents, plan mode
5. Built for .NET — Roslyn integration gives "real compiler intelligence: type hierarchies, call graphs, cross-assembly references"
6. LSP for C# and TypeScript — hover, go-to-definition, references
7. Playbooks — "typed, composable, stateful workflow automation" with step sequencing and gates
8. Dual-box mode — works behind NAT/CGNAT via relay
9. Persistent sessions — JSONL transcripts, cross-session memory, snapshot-based file undo

## Performance Benchmarks (tok/s)

| Hardware | Model | tok/s |
|---|---|---|
| Ryzen 9 7940HS (CPU) | Qwen3.6 35B A3B | ~17–20 |
| Apple M5 Pro (64GB) | Qwen3.6 35B A3B | ~45–48 |
| RTX 3090 (~$700 used) | Qwen3.6 27B | ~42–45 |
| RTX 4090 | Qwen3.6 27B | ~47–50 |
| RTX 5090 | Qwen3.6 27B | ~75–80 |
| RTX 3060 (12GB) | Qwen3.5 9B | ~38–40 |

The site calls a used RTX 3090 "indistinguishable from a cloud API" and notes a NUC with 32GB RAM delivers "a solid 20 tok/s."

## Installation

```
bash <(curl -fsSL https://raw.githubusercontent.com/StartupHakk/OpenMonoAgent.ai/refs/heads/main/get-openmono.sh)
```

Then run: `openmono agent`

## Pricing

"Unlimited tokens. Forever." — Free, no per-token billing, no rate limits, no subscriptions.

## Comparison

Positioned as the "third option" vs. Claude Code and OpenCode: fully offline privacy, Docker-native sandboxing (vs. host install for competitors), C#/.NET (vs. npm), free unlimited tokens (vs. per-token pricing).

## Links

- GitHub: github.com/StartupHakk/OpenMonoAgent.ai
- Parent company: startuphakk.com
- Mobile app: iOS App Store and Google Play
- Playbooks, Isolation pages on the site
