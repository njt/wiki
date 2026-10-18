---
url: https://github.com/statico/ttmux
title: "ttmux — a modern multiplexer alternative"
author: statico
date_fetched: 2026-10-18
date_published: 2026
topics:
  - developer-tools
  - agent-coding-workflow
---

ttmux is a Rust terminal multiplexer (macOS/Linux, ~22K lines across 24 modules) that positions itself as a modern alternative to tmux and screen: floating panes, real mouse support, curved borders, an in-app settings screen, and — unusually for a mux — first-class awareness of coding agents. It is MIT-licensed, beta, installable via Homebrew or `cargo install`.

The architecture is a client-server split in the tmux tradition: a server owns the PTYs and the `App`; clients are just viewers over a length-prefixed JSON protocol on a unix socket (`src/proto.rs`), with per-client bounded queues so one stalled attachment can never freeze the session. `ttmux upgrade` migrates a live session to a new server binary, which doubles as the fix for macOS keychain/permission prompts tied to the spawning terminal.

Two things make it notable beyond the mux genre. First, `src/agent.rs` implements a per-pane `Watcher` classifier (Idle / Busy / Attention / Done) from configurable regex patterns over the pane's tail plus OSC 7501 program-status reports, with a 1.5s grace timer so spinner flicker doesn't misclassify; the status bar counts panes waiting on you and a keybinding jumps to the next one. Second, the README states plainly that the entire project was written by Claude Code — a ~22K-line Rust codebase as an artifact of agentic development, with an `ARCHITECTURE.md` that reads like a contract doc for a maintainer who must trust generated code.
