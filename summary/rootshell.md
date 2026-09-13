---
url: https://github.com/kitknox/rootshell
title: "rootshell"
author: Kit Knox
date_fetched: 2026-09-13
date_published: 2026-09-12
topics:
  - developer-tools
  - agent-coding-workflow
---

rootshell is a free, MIT-licensed terminal emulator for iPhone, iPad, Vision Pro, and Mac by Kit Knox — ~460K lines of Swift across ~1,300 files, built on libghostty for Metal-accelerated rendering. It is a terminal in the way a Swiss Army knife is a knife: native SSH (Citadel/NIOSwift with post-quantum key exchange, Secure Enclave and YubiKey keys), a from-scratch Swift mosh implementation (Rootshell Roam), tmux control mode, HEVC screen sharing, a git client over libgit2, a Helix editor port, a WASI Preview 1 sandbox for running user-compiled CLI tools on iOS, and a built-in multi-provider AI agent with voice mode.

What makes it interesting for agent practice is the Agent Inbox: the terminal detects coding agents (Claude Code, Codex, Cursor, Copilot, OpenCode, Antigravity, Pi) on-device by scraping the rendered terminal screen with a rules engine ported from herdr — no server-side install — and shows each agent as a live status card, routes push notifications when one needs input, and can deep-link back to the exact pane inside tmux. Paired computers can send end-to-end-encrypted (HPKE X-Wing) notifications from CLI hooks, including Claude Code and Codex turn/permission events.

The engineering is unusually deep for a mobile app: wrap-tolerant and multiplexer-aware screen detection, OCB-encrypted mosh state sync with hardware AES, a stateless post-quantum push relay, and an unsandboxed macOS helper that validates clients by code signature before passing PTYs over an App Group socket. The code reads as pragmatically vibe-coded: extensive reasoning in doc comments and a real security model, but also loose edges like two parallel SSH session implementations and a README that calls the SSH stack "no external dependencies" while the app ships ~20 forked Swift packages.
