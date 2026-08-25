---
title: "We Built a Scalable Agent Sandbox"
url: https://newsletter.cloudsquid.io/p/we-built-a-scalable-agent-sandbox
author: CloudSquid
date_fetched: 2026-08-25
---

# We Built a Scalable Agent Sandbox

A CloudSquid newsletter post describing how the team scaled agent-driven workflows across enterprise data — PDFs, CSVs, Excel, and SFTP exports from legacy systems with no APIs — without giving up governance and access controls.

## The Problem

A single agent works fine in a single workflow. At 100 agents running in parallel against the same enterprise data, structural limits appear: agents overwrite each other's work, produce conflicting results, and — in a recent Anthropic multi-agent study the author cites — even start "turf wars."

## Three approaches, then a virtual file system

- **v1 — State machine.** A graph of every possible user action, with the agent choosing from a small pre-defined response set at each step. Tightly controlled, but it hit a ceiling: double build (once as UI for humans, once as a rigid graph for the agent), un-anticipatable edge cases, and model improvements locked away behind the constraint.
- **v2 — SQL access.** Agents queried structured extraction tables directly instead of navigating a fixed graph. Left PDFs, emails, and unstructured documents still out of reach in enterprise implementations.
- **v3 — Sandboxed terminal.** Agents ran scripts, queried databases, and read/wrote files like Claude Code or Codex. Worked, but hit Anthropic's hosted-terminal file-count ceiling — a sensible anti-abuse limit that a finance team juggling hundreds of documents per task can't live with.

The final design: one isolated file store (Google Cloud), and for every agent task an isolated, permissioned copy of only the files that task may touch. Each agent reads what it's permitted to read, writes what it's permitted to write, and can't see what's hidden. On completion, changes sync back to the true source in a controlled, ordered way — no race conditions, no silent overwrites, fully auditable. Agent count is decoupled from collision risk, because collision was never possible.

## The counter-intuitive payoff

Sequential hand-offs — "Agent A must finish before Agent B starts" — turn out to be almost never necessary. With the workspace configured correctly upfront (Standard Operating Procedures), a hundred agents complete a reconciliation task in a single pass with no coordination overhead.
