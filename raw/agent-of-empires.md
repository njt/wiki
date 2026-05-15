---
title: "Agent of Empires"
url: https://github.com/njbrake/agent-of-empires
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Agent of Empires (AoE)

Session manager for AI coding agents on Linux and macOS, enabling users to run multiple AI agents in parallel across different repository branches with isolated tmux sessions and optional Docker sandboxing.

## Key Features

**Multi-Agent Support:** Claude Code, OpenCode, Mistral Vibe, Codex CLI, Gemini CLI, Cursor CLI, Copilot CLI, Pi.dev, Factory Droid, Hermes, Kiro CLI, Qwen Code.

**Interface Options:**
- Terminal User Interface (TUI)
- Web dashboard (Beta)
- Cockpit (Alpha) — mobile-first native rendering with plan panels and swipe-to-approve
- CLI

**Session Management:** Persistent tmux sessions that remain active after closing the app.

**Remote Access:** Press `R` in TUI to expose web dashboard over HTTPS with QR code and passphrase auth (Tailscale Funnel or Cloudflare Tunnel).

**Git Integration:** git worktrees for parallel agent operation on different branches, multi-repo workspaces.

**Technical Stack:** Rust (77.3%), TypeScript (19.7%), tmux for persistence.

## Metrics
- 2.2k GitHub stars, 187 forks, 90 releases
- Latest: v1.6.2, MIT License
