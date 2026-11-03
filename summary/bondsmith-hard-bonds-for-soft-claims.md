---
url: https://coldtake.dev/blog/bondsmith-hard-bonds-for-soft-claims
title: "Bondsmith: Hard Bonds for Soft Claims"
author: coldtake.dev (unnamed author)
date_fetched: 2026-11-03
date_published: 2026 (recent; exact date not stated)
topics:
  - guardrails-and-feedback-loops
  - agent-coding-workflow
---

Bondsmith is a Rust workflow engine for agentic development built around a single rule: no claim counts without a proof bonded to it. Whatever an LLM says about the code it wrote ("all tests pass", "I committed the fix") is a *soft claim*; the engine only accepts work when a deterministic check — a Gate run by the Driver, never by the agent — proves the step's Postconditions hold.

The engine distinguishes three roles: a **Worker** is a disposable headless LLM session that edits files and commits, and whose self-reports nobody asks about; a **Gate** is a deterministic check (lint, type-check, tests); and the **Driver** is a state machine that spawns Workers, runs Gates, and is the only thing allowed to write state. Postconditions are evaluated in plain Rust over facts gathered from disk, git and Gate results, with a closed verb vocabulary (`exit_zero`, `sha_is_ancestor`, `non_empty`, `gate_pool_pass`) and fail-closed semantics: a missing fact fails the check.

Flows (feature delivery, issue fixing) are plain data — JSON contracts and Markdown prompts read from disk — so workflows can be edited without touching the engine. Nothing in a Flow references a specific harness; Workers are launched through one port, with Claude Code implemented today and Pi next. Every run leaves a state file, an append-only event log, and per-Worker session transcripts, surfaced in a read-only local UI.

The post closes with a worked example: four of five Postconditions on a test-first step held, yet the run was held because one test still failed — and with a takeaway that survives without the tool: pick one action your agent reports on and replace the report with a check a program runs. Bondsmith is going open source "in the upcoming weeks."
