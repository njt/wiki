---
url: https://blog.lhotka.net/2026/09/21/Remote-Control-Server
title: "Remote Control Server"
author: Rockford Lhotka
date_fetched: 2026-09-23
date_published: 2026-09-21
topics:
  - claude-code
  - agent-coding-workflow
---

Rockford Lhotka continues his series on driving Claude Code from remote machines. His previous SSH setup worked but died whenever the connection dropped, so he moved to Claude Code's Remote Control: `/rc` in an already-running session exposes it to the Claude mobile/desktop apps and browser. The real prize is `claude remote-control`, a long-lived server that can spawn *new* sessions on demand — no one has to be at the machine first.

He details how to run it headless on Windows via a scheduled task (S4U logon, boot trigger plus a 15-minute self-heal trigger, no execution time limit), including one-time setup steps (login and workspace-trust acceptance) because a background logon can't show prompts — and can't use Credential Manager, which breaks secret-holding tools. The "nobody is logged in" problems from SSH persist: Docker Desktop needs a login, and Git/`gh` need non-prompting credentials.

Operationally, the server exits after ~10 minutes of network loss, and background updates never take effect until restart — so his supervisor script stages `claude update` hourly and restarts when a new version is staged *and* transcripts under `~\.claude\projects` have been idle an hour. He notes the `Capacity: 1/32` display is misleading (the server keeps one pre-created session), and that a script can only restart a server it started itself because each scheduled-task run is a separate background logon.

The payoff, his "airplane test": all model traffic runs over the office connection; only the chat traverses airplane wifi. Sessions run on his own machines with his full toolset (Docker, kubectl, .NET SDKs, MCP servers) rather than a generic cloud sandbox.
