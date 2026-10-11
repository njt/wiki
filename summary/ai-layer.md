---
url: https://gitlab.com/kriscodes2-claude-plugins/ai-layer
title: "Bootstrap AI Layer (ai-layer Claude Code plugin)"
author: Kris Codes LLC
date_fetched: 2026-10-11
date_published: 2026-09-07
topics:
  - claude-code
  - coding-agents-and-frameworks
---

A README for **ai-layer**, a free MIT Claude Code plugin from Kris Codes LLC that gives a session persistent memory and code intelligence: 88 skills and 25 agents across three packs (core, marketing, infra), built on two MCP servers — `duckdb-brain` (a DuckDB "brain" that remembers across restarts) and `typescript-lsp` (symbol/call-site lookup so the agent doesn't read whole files to find who calls a function).

The interesting engineering is the **three-mode brain resolver**: `gateway` (a commercial Koinessa Brain Gateway — not included), `local` (single-machine DuckDB file), or `setup` (deps not yet installed). Precedence is deliberate and *not* a fallback: a configured-but-unreachable gateway stays in gateway mode and fails open with a named error rather than silently downgrading to a different brain. Setup mode is equally explicit — one status tool per server that points at `/ai-layer:doctor --fix`, so nothing degrades quietly. One command (`doctor --fix`) does the whole install because the native DuckDB binding and the spawned language server can't be bundled into a git-cloned plugin.

The lifecycle is the other half: init → create-prd → plan-review → quality-gate → ci-local → fix-ci → ship → security-review, plus session memory commands (recall/remember/resume-work) and a retrieval-first SOP that routes symbol lookups through the LSP index instead of grep. The README even ships a test (`tests/docs/readme.test.mjs`) that checks the architecture diagram against the resolver, so a fourth mode fails CI rather than shipping a stale doc. Privacy is a stated design constraint: local mode makes no network connections; gateway mode talks to exactly one host you configure; no telemetry in any mode. Distribution is a GitLab git URL — releases are one-commit snapshots of a private monorepo, with merge requests re-applied upstream rather than merged.
