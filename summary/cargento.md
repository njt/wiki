---
url: https://github.com/spacedock-dev/cargento
title: "Cargento — Agnostic Agent Cartography and Visualization"
author: spacedock-dev
date_fetched: 2026-08-01
date_published: 2025
topics:
  - developer-tools
---

Cargento is a local web dashboard that passively maps live coding-agent activity
across nine AI coding tools: Claude Code, Codex, Pi, Gemini CLI/Antigravity
CLI, GitHub Copilot CLI, OpenCode, Cursor CLI, Goose, and Factory Droid. It
reads each tool's local session stores read-only, assembles a self-refreshing
HTML dashboard, and fires desktop notifications when a Claude session is blocked
waiting for human input.

The project is a single Python plugin distributed across four harness
marketplaces. It is stdlib-only with no pip dependencies, requiring Python
3.11+. The runtime decomposes into ~25 modules with explicit inward-only
dependency direction, enforced by an AST-based import graph test.

Key architectural choices: frozen `RuntimeConfig` built once at process
boundary, mutable `RuntimeState` for caches and scanner offsets, and an
`Application` class binding both to injected services (notifiers, clock,
diagnostic sink) — so two servers can run in one interpreter without shared
state.

Each harness collector runs inside its own `try/except Exception` boundary: one
corrupt transcript or SQLite database can only take down its own harness, never
the whole dashboard. Timestamps implausibly ahead of the clock are rejected
rather than clamped to zero, preventing a clock-skewed restored session from
reading as perpetually active. Turn scanning is incremental (reads only bytes
appended since last call) with gap-detection logic that re-anchors elapsed time
to post-gap events, so idle stretches don't inflate duration estimates.

The dashboard has two display modes (card stack and calm ledger), keyboard
navigation, per-session ETA estimates, subagent tracking with named pills, and
rate sparklines. Collection memo with lock-held-through-collection prevents
concurrent requests from stampeding cold cache entries.

Loopback enforcement uses three header checks (Host, Origin, Sec-Fetch-Site) to
defeat DNS rebinding and cross-site fetch. On Windows, `SO_EXCLUSIVEADDRUSE`
prevents port hijacking. All harness stores are read-only and no data leaves the
machine.

Cargento sits in the observability layer — it reads what already exists rather
than asking for integration. It is a companion to [[Subspace]] (same
organization, same store-adjacent philosophy) and fills the runtime visibility
role that [[AgentsView]] also targets, but with deeper per-session state
reconstruction and multi-marketplace plugin distribution.
