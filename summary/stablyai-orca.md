---
url: https://github.com/stablyai/orca
title: "Orca"
author: stablyai
date_fetched: 2026-08-01
date_published: 2025
---

Orca is a cross-platform Electron desktop app that acts as an AI orchestrator — it runs multiple CLI AI coding agents (Claude Code, Codex, OpenCode, Pi, Grok, and 20+ others) in parallel git worktrees, with a React Native mobile companion for monitoring and steering agents from a phone. It's a meta-tool: it doesn't implement AI models but provides the environment, coordination, and persistence layer for running existing CLI agents at scale. The codebase is substantial — ~590,000 lines of TypeScript across ~9,500 source files, MIT licensed.

The most architecturally significant decision is **persistent PTY via a forked daemon**: instead of hosting terminal sessions in the main Electron process, Orca forks a child daemon that owns the actual pseudo-terminals. This lets terminal sessions survive renderer crashes, window close, and even main process restart. Paired with that is an **agent hook server** — a local HTTP server that receives structured status pings from hooks installed into each agent's runtime, giving Orca reliable state tracking (idle → working → blocked → done) without parsing terminal output. A **custom SSH multiplexer** built on `ssh2` funnels PTY, SFTP, git, file watching, and agent hook channels over a single TCP connection, with WSL awareness and auto-reconnect.

The central orchestrator (OrcaRuntimeService) manages the live graph of worktrees, terminals, and browser panes, dispatching tasks via a coordinator that can decompose work and fan out to worker agents. A separate orchestration database (SQLite) tracks runs, tasks, and message history, while core settings and metadata use a simpler JSON file store — a split that suggests organic growth.

Agent integrations follow a shared pattern (account service, hook installation, session resume) but with significant per-agent code duplication. The most complex is Codex, with multi-faceted session resume logic spanning CODEX_HOME management, legacy migration, and trust pre-marking. Other notable techniques include synthetic title injection for agents that don't natively emit status, bracketed-paste-mode prompt injection to avoid slow line-by-line typing, and extensive crash recovery with renderer reload backoff, GPU fallback, and crash breadcrumbs.

Compared to peers: Broomy is the closest analogue (another MIT-licensed Electron multi-agent runner) but narrower in scope. Traycer adds Yjs-based real-time collaboration. cmux is a simpler native-macOS approach. Fleet Supervisor is a Python alternative with pluggable backends. Orca distinguishes itself with SSH-remote worktrees, mobile companion, orchestration engine, computer-use desktop automation, and a plugin system — at the cost of significant complexity.
