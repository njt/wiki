---
url: https://github.com/paperclipai/paperclip
title: "Paperclip"
author: Paperclip Labs, Inc.
date_fetched: 2026-09-11
date_published: 2026
source_type: GitHub repository
---

Paperclip is an open-source (MIT) "control plane" for running teams of AI agents as a company. Where an individual coding agent (OpenClaw, Claude Code, Codex, Cursor) is an "employee," Paperclip is the "company": it supplies the org chart, goals, budgets, governance, approval gates, and coordination that make many agents produce work rather than collide. It is a Node.js server plus a React UI that schedules agents on heartbeats, hands them tickets, and tracks their output and spend from a task-manager-style dashboard.

The core abstraction is deliberately thin: an agent is a database row (`adapterType` + JSON config) that can be driven by any of a dozen adapters — process CLIs (claude, codex, cursor, gemini, grok, kimi, opencode, pi), HTTP/webhook bots (OpenClaw, Hermes), cloud harnesses (Cursor Cloud), and a bundled Rust "runner" for native Codex/OpenCode/ACPX execution. "If it can receive a heartbeat, it's hired." Each adapter normalizes its output into a shared provider-neutral transcript, and the server reads the agent's `run_result` disposition (done / blocked / needs_review / yielded) to decide whether the work is finished.

The engineering centerpiece is the wake/heartbeat execution model: tasks hold an atomic execution lock, new wakes are coalesced into a running run, deferred behind it, or promoted one-at-a-time after it releases, with automatic recovery runs re-queued for stranded or failed tasks. All of this runs inside Postgres transactions, with a pure-function policy layer (facts in, decision out) kept separate from the database reads and writes. It ships as a pnpm monorepo with an embedded Postgres (Drizzle ORM, 135 tables, 273 migrations) so a single process can run a whole company locally, or be pointed at external Postgres for production.
