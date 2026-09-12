---
url: https://www.warp.dev/blog/introducing-the-warp-agent-cli-coding-agent
title: "Introducing the Warp Agent CLI: a CLI coding agent that does what others can't"
author: Warp
site: warp.dev
date_fetched: 2026-08-06
topics:
  - coding-agents-and-frameworks
  - agent-orchestration
---

Warp launched a standalone CLI version of their multi-model coding agent, decoupling it from the Warp Terminal app so it can run in any terminal (Ghostty, iTerm 2, VS Code, Windows Terminal). The CLI is built on Warp's terminal infrastructure, giving it a unique multiplexing architecture that manages PTY connections with a layer of indirection between the agent and the underlying shell — similar to how tmux works.

Three architectural differentiators: (1) **Terminal mux'ing** enables persistent sessions across directory changes and remote machines without installing binaries, agent-driven control of full-screen interactive apps (sqlite, python REPLs, gdb, htop), and natural-language command detection with tab completion. (2) **Agent orchestration** is built in — the Warp Agent is an orchestrator that delegates to subagents, with a native UI for switching between orchestrator and subagent sessions, plus cloud agent handoff for continuing work after closing a laptop. When coupled with Warp's cloud platform, the harness can delegate across different harnesses like Claude Code and Codex, not just different models. (3) **Cost-optimizing harness** with auto-routing based on task complexity, access to frontier and open-weight models, and support for custom model routers.

Pricing starts at $18/month ($20 inference credit), with ad-hoc credits from $10 and BYO API key / OpenAI-compatible endpoint / SuperGrok options. The CLI is available for Mac, Linux, and Windows.
