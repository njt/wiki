---
title: "kata"
url: https://github.com/wesm/kata
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - developer-tools
  - agent-coding-workflow
---

Local-first issue tracking for AI-assisted software work by Wes McKinney. Agent-friendly CLI and human-facing TUI.

Gives agents a structured place to record tasks, decisions, links, comments, and state changes without turning GitHub Issues, markdown plans, or chat transcripts into the source of truth.

CLI built for agents and automation: stable commands, JSON output, predictable failure modes. TUI built for people: browse, triage, edit, supervise agent-written work without reading raw JSON. Both talk to the same local daemon and SQLite database.

Key design: workspace-to-project binding via .kata.toml. Data lives locally in SQLite under KATA_HOME behind a long-running daemon. Issues have stable ULID uid values. Short IDs derived from ULID for display.

Goals: agent ergonomics (stable commands, JSON-first, idempotency keys, predictable exit codes), human oversight (TUI for browsing and supervising), auditability (append-only comments, event history, actor attribution).

Compared to Beads (Dolt-powered, large capability system), kata makes a different architectural bet: the issue ledger should be a local service adjacent to workspaces, not a database owned by each repository. Deliberately smaller: one daemon, one local store, one HTTP API, one TUI, and a narrow issue model.