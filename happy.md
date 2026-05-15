# happy

Mobile and web client for Claude Code and Codex with realtime voice, end-to-end encryption, and full features. Run `happy` instead of `claude`, then monitor and control your coding agent from your phone. Device switching with a single keypress, push notifications for permissions and errors, fully open source with no telemetry.

---

## Key Quotes

> (No notable quotes from the source -- the README is functional rather than philosophical.)

## Key Themes

#developer-tools #mobile #claude-code #encryption

Happy solves the "I started a Claude Code session and now I need to leave my desk" problem. The core components are the CLI wrapper, a mobile app (Expo-based, iOS + Android), a web interface, and a backend for encrypted synchronization.

At 20.6k stars, this is one of the most popular tools in the Claude Code ecosystem. The appeal is obvious: coding agents run long tasks, and being tethered to your terminal while they work is a friction point that mobile access eliminates.

Connects to [[vibes-cli]] (both are about making Claude Code accessible outside traditional terminals) and [[engineering-notebook]] (which also deals with the problem of maintaining visibility into what your agent is doing when you're not watching).

## Critical Analysis

The encryption story is the make-or-break for this tool. Your code passes through Happy's backend for synchronization -- if the end-to-end encryption is solid, this is a genuine quality-of-life improvement. If it's not, you're streaming your codebase through a third-party server. The open-source nature helps with auditability but doesn't guarantee the crypto is correctly implemented. For personal projects: obviously useful. For corporate code: requires a security review that most teams won't do.

---
*Sources: [[raw/happy]]*
*Last updated: 2026-05-14*
