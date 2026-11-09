---
url: https://github.com/pingdotgg/t3code
title: "T3 Code — agent harness control surface"
author: pingdotgg (Theo Browne et al.)
date_fetched: 2026-11-09
date_published: 2026 (repo, very early)
topics:
  - coding-agents-and-frameworks
  - agent-orchestration
---

T3 Code is an open-source "agent harness control surface" from Ping (the t3.gg team): a server that runs on your machine and controls existing coding agents — Claude Code, Codex, Cursor, Grok Build, OpenCode, Google Antigravity, and fx — from iOS/Android apps, a web app, and an Electron desktop app. It reuses your existing provider subscriptions; the tool is a layer on top of the agent CLIs, not a new harness.

The architecture docs reveal a serious distributed system underneath the friendly product framing. A server owns the workspace (provider processes, terminals, Git, files) while independently versioned clients control it over an authenticated RPC contract built on Effect's RPC/Schema. Orchestration-v2 is an event-sourced design: a single ~14.7k-line Orchestrator serialises commands and decides domain events without doing side effects; an EventSink commits events, projections, command receipts, and outbox effects in one transaction, and an EffectWorker performs provider/filesystem work afterwards. The event log retains only about a day for replay and idempotent retries, with ProjectionMaintenance pruning events the projections already cover.

Remote access is a first-class product: direct LAN, Tailscale, SSH, and a hosted relay ("T3 Connect", Clerk for identity, Cloudflare Workers + Durable Objects for webhook holding) all just change the route to the same environment; identity is independent of the route, and sessions use DPoP-bound tokens with live socket renewal. Providers sit behind per-driver adapters with per-instance isolation, checkpoints use hidden Git refs so no commits land on the user's branch, and thread settlement (PR detection, auto-settle) runs server-side without a connected client. The repo even ships model-driven evals for its own MCP tool descriptions and thread titles.

Positions itself as a fork-friendly alternative to Conductor, Codex desktop, Claude Desktop, and Cursor Glass — explicitly "we're selling nothing," with very early-stage caveats and mostly closed to contributions.
