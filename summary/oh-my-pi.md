---
url: https://github.com/can1357/oh-my-pi
title: "oh-my-pi (omp) — A Coding Agent with the IDE Wired In"
author: Can Bölük (fork of Mario Zechner's pi-mono)
date_fetched: 2026-07-11
date_published: 2025
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

oh-my-pi (omp) is an open-source, terminal-first coding agent harness — a fork of
Mario Zechner's Pi rewritten as a batteries-included developer surface. It spans
~345K lines of TypeScript (Bun runtime) and ~55K lines of Rust (N-API addon),
bundling 32 built-in tools, 40+ LLM providers, a TUI, memory systems, subagent
orchestration, browser automation, and collaboration support.

Its core thesis is that harness correctness matters more than model intelligence:
in-process native tools (ripgrep, bash, tree-sitter), content-hash-anchored
editing (Hashline), and time-traveling stream rules (TTSR) eliminate whole
classes of editing and tool-calling errors upstream of the model. The README
cites Grok Code Fast 1 jumping from 6.7% to 68.3% pass rate purely by fixing
the edit format.

Architecturally, omp layers agent orchestration, session management, and TUI
rendering in TypeScript over a Rust native addon that runs grep, shell, AST
analysis, and token counting in-process — no fork/exec on the hot path. Sessions
are append-only JSONL with tree-shaped branching. Two independent memory
systems (Hindsight markdown pipeline and Mnemopi SQLite+vector store) feed
context into future sessions.

The project makes deliberate trade-offs: monorepo complexity (14 npm packages +
6 Rust crates) for tight integration, a Rust native layer for portability
across platforms without binary dependencies, and configuration inheritance from
eight competitor formats for adoption ease. Notable weaknesses include tool
sprawl (32 tools for the model to navigate, mitigated by BM25-indexed
discovery) and compaction-system complexity with six trigger paths.
