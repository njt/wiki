---
url: https://domenic.me/agentic-coding-setup/
title: "My Agentic Coding Setup, July 2026"
author: Domenic Denicola
date_fetched: 2026-07-25
date_published: 2026-07-23
---

Domenic Denicola describes his AI-assisted coding setup as of July 2026, built around a
Linux VM, Tailscale networking, and dual agent harnesses (Claude Code and Codex CLI).

The core is an Ubuntu Server VM running on his always-on home desktop (4 cores, 8 GB
RAM). Tailscale connects his desktop, laptop, phone, and VM into a private network, with
Tailscale SSH eliminating the need to manage SSH keys. He accesses agents primarily
through the ChatGPT app over SSH, which he praises for handling disconnects and
backgrounding gracefully — unlike Claude Code's desktop app, which kills sessions when
the client closes.

He runs agents with no approval prompts (`--dangerously-skip-permissions`,
`--yolo`) and full sudo access, arguing that aligned models aren't malicious and the real
risk is agentic overaction (like merging a PR prematurely). Git worktrees enable parallel
agents, each in their own directory and branch, without file or port conflicts.

For code review, he uses VS Code's Remote-SSH extension connected to the VM. For dev
servers, a wrapper around Portless provides HTTPS preview URLs on the tailnet, meeting
secure-context requirements for Service Workers and web APIs.

Configuration (dotfiles, AGENTS.md, skills) is synced across machines with chezmoi, kept
in a private GitHub repo. His phone-based workflow is a natural extension: spot a bug on
the train, ask the agent to fix it via ChatGPT mobile over Tailscale, and merge the PR
before arriving home.

Gaps he identifies: better dev-container isolation, session history tied to folder paths
(renaming a project loses history), and automatic transcript backups to a durable store.
