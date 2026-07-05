---
url: https://github.com/StartupHakk/OpenMonoAgent.ai
date_fetched: 2026-07-05
backfilled: true
---

**Open-source coding agent. Local-first. Zero cost. Zero cloud.**

Built to democratize AI. Powered by .NET.

OpenMono is a coding agent that runs **entirely on your hardware** — no subscriptions, no data leaving your network, no per-token billing. It pairs a .NET 10 CLI with its own llama.cpp inference server, giving you a full agentic loop with **20 built-in tools**, Docker sandboxing, and deep code intelligence. NVIDIA GPU, CPU, or Apple Silicon (Metal) — **it auto-configures itself**. You own the model, the compute, and the data.

One command. Auto-detects GPU · CPU · Apple Silicon. Installs model, runtime, and Docker containers.

`bash <(curl -fsSL https://raw.githubusercontent.com/StartupHakk/OpenMonoAgent.ai/refs/heads/main/get-openmono.sh)`Then from any project:

```
openmono agent            # TUI mode (default)
openmono agent --classic  # classic scrolling terminal
```
→ Full command reference — daily commands, setup flags, GPU/CPU options

**Get them:** `openmono setup search` · `OPENMONO_VISION_ENABLED=1` · App Store · Google Play · VS Code Marketplace

Most coding agents are cloud products wearing an open-source label. Your prompts, your code, and your context hit someone else's servers on every keystroke. OpenMono runs the model on your hardware — after the one-time setup, **inference costs nothing**. Your code never leaves the machine. No account. No usage dashboard. No API key.

→ Full architecture + diagram · 4 providers · runs at **~45 tok/s on GPU**, ~20 tok/s on CPU

| 
 
 | 
 | 
| 
 | 
 
 | 
| 
 | 
 Auto-detects graphify (semantic concept graph, 25+ languages) and code-review-graph (structural call graph via MCP, ~22 tools) if installed — no config needed. | 
| 
 | 
 | 
| 
 | 
 | 
| 
 
 | 
 
 | 

Auto-detects GPU · CPU · no config needed. On Linux, 12 GB and 16 GB cards run lower-accuracy models; use a 24 GB card for best results. Requires Ubuntu 26.04 LTS (recommended) or 25.10. On macOS, the full and inference roles require Apple Silicon (M1+); 64 GB+ unified memory is the recommended, tested configuration. Less than 64 GB is not encouraged — smaller model, much tighter context window. Intel Macs: agent-only mode. macOS 14+ (Sonoma/Sequoia) recommended.

| `Roadmap` | What's next | 
| `Setup & commands` | Daily commands, TUI vs classic, flags | 
| `Architecture` | .NET CLI + llama.cpp + Docker, full diagram | 
| `Models & reasoning` | Model tiers, reasoning mode, provider config | 
| `Configuration` | settings.json, providers, permissions, MCP servers | 
| `Playbooks` | YAML workflows, typed params, checkpoint/resume | 
| `graphify` | Semantic code graph, 25+ languages | 
| `code-review-graph` | Structural call graph via MCP | 
| `VS Code extension` | Chat panel for VS Code 1.85+ · also works in Cursor · Marketplace | 
| `Contributing` | How to contribute | 

| PUBLIC BETA Early access is open — we're shipping updates fast. Try it out and tell us what you'd like to see next. | 

| OpenMono is early and moving fast. Contributions are welcome — new tools, providers, LSP servers, playbooks, bug fixes, or docs. | 
| `→ Read the contributing guide before opening a PR` | 

*"AI shouldn't be a subscription you rent. It should be infrastructure you own —*

sitting on your desk, serving your code, answering only to you."

sitting on your desk, serving your code, answering only to you."

— Startup Hakk
