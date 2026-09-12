---
title: "Claude Code on the Go"
url: https://granda.org/en/2026/01/02/claude-code-on-the-go/
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-coding-workflow
  - personal-agents
---

# Claude Code On-The-Go

## Setup
- Vultr VM (8-core, 32GB RAM) at $0.29/hour, started/stopped on-demand
- Tailscale VPN for secure, private network access
- Termius SSH client + mosh for resilient mobile connections
- tmux for persistent session management
- Poke service for push notifications when Claude needs input

## Notable Quotes

"No laptop, no desktop—just Termius on iOS and a cloud VM."

"The loop is: kick off a task, pocket the phone, get notified when Claude needs input."

"Without notifications, you'd constantly check the terminal. With them, you can walk away."

"Six agents, six features, one phone."

## Key Workflow
Asynchronous development from anywhere:
1. Automate VM lifecycle via CLI scripts and iOS Shortcuts
2. Notifications triggered by Claude's input requests
3. Parallel Claude agents using git worktrees and deterministic port allocation
4. Security via Tailscale isolation and cost controls

Development integrated into daily gaps rather than requiring dedicated desk sessions.
