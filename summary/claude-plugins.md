---
url: https://github.com/erikdarlingdata/claude-plugins
title: "Darling Data — Claude Code plugins"
author: Erik Darling (Darling Data)
date_fetched: 2026-09-20
date_published: 2026-09-19
topics:
  - claude-code
  - guardrails-and-feedback-loops
---

Erik Darling's plugin repo, distributed three ways at once: a Claude Code plugin marketplace, a GitHub Copilot CLI plugin, and a pi package — all three harnesses reading the same `SKILL.md` format, so one skill source ships natively to three agents with no conversion layer. The flagship is `sqlserver-query-plans`, a model-invoked skill that teaches an agent to read a SQL Server execution plan and say what is actually slow and why, built explicitly out of "the conclusions that sound authoritative and are wrong": cost percentages read as measurements, operators ranked by cumulative time, `EstimateRows` compared to a total `ActualRows`, missing-index requests pasted as DDL, scans condemned on sight.

The skill's load-bearing component is `scripts/extract.py`, 1,299 lines of stdlib-only Python that flattens a `.sqlplan` into a compact text digest. This is not a convenience wrapper: `.sqlplan` files are UTF-16 (so `grep` silently matches nothing), some lie about their own encoding, real plans run to megabytes, and the raw counters cannot be quoted safely without a corrected measurement model. The extractor computes self-time attribution (row-mode times are cumulative; batch mode is standalone; exchanges lie; parallel plans must subtract within a thread, never across), normalizes cardinality per execution, fingerprints the optimizer's fixed-guess selectivities, and hardens its own output against prompt injection carried in plan strings — a `.sqlplan` is treated as a file an attacker handed you.

Two pi-only extensions round it out. `pi-session-resume` registers every interactive pi session and reopens the ones killed by reboot or crash — as Ghostty tabs or tmux windows — distinguishing a deliberate quit from a signal-driven death by prepending its own SIGTERM listener. `pi-subagent-watchdog` fills the gap where background subagents only report at completion: it polls each running child's vitals (tokens, context %, tool uses, turns, wall clock, compactions), and when a threshold crosses, injects a numbered check-in into the orchestrator with a decision protocol — guide mode assesses with judgment, strict mode treats thresholds as budgets. A 754-line pi setup guide for Claude Code expats sits alongside, into which the plugins slot as §11–13.

*Sources: raw/claude-plugins.md (verbatim README); repo analyzed at commit of 2026-09-19, plugin version 1.5.4.*
