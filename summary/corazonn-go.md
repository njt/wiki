---
url: https://github.com/schuyler/corazonn/blob/main/.claude/commands/go.md
title: "TDD Coordinator (go.md)"
author: schuyler
date_fetched: 2026-05-14
date_published: unknown
topics:
  - agent-orchestration
---

# TDD Coordinator (go.md)

A Claude Code slash command (`/go`) that orchestrates subagents through a complete test-driven development workflow: red/green/refactor cycles with a mandatory "Rule of Two" quality gate where every work product is reviewed by a different agent before it counts as done.

## Core Design

The coordinator never writes code or tests directly — it only coordinates subagents via TodoWrite. Six principles:

1. **Delegation** — coordination only, no direct implementation
2. **Organization** — TodoWrite for tracking all work
3. **Rule of Two** — every output reviewed by a different agent against Quality, Correctness, and Adherence criteria
4. **Objective completion** — verified by testing, not by assertion
5. **Parallel execution** — only for independent tasks (multiple Karl+Zeppo pairs on different components)
6. **No thrashing** — ask user for help after 4-5 failed attempts

## Agent Roles (Marx Brothers)

- **Groucho** — analyzes requirements in `docs/`
- **Chico** — gates understanding, reviews completed implementations
- **Karl** — writes failing tests (red), then implements (green)
- **Zeppo** — gates Karl's tests, then gates Karl's implementation
- **Harpo** — writes documentation for completed features

## Five-Phase Workflow

1. **UNDERSTAND** — Groucho analyzes requirements; Chico or Karl gates the understanding
2. **IMPLEMENT** — Karl: red (failing test) → Zeppo gates test → Karl: green (implementation) → Zeppo gates implementation → loop until approval
3. **REVIEW** — Chico reviews complete implementation; Groucho or Zeppo validates Chico's review; issues loop back to Karl
4. **DOCUMENT** — Harpo writes docs; Chico or Karl gates
5. **VERIFY COMPLETION** — all gates pass; tasks.md updated; uncertainty flagged

## Rule of Two Detail

The reviewer checks three criteria:
- **Quality** — is it well-written?
- **Correctness** — does it do what it should?
- **Adherence** — does it follow the PRD/TRD/design docs?

Critical issues MUST fix before proceeding; minor issues may proceed at discretion.

## Optional Argument

`/go [context]` where context can be: task name/ID, path to specific PRD/TRD file, or continuation marker from a previous session. Without an argument, searches `docs/` for requirements.

## Opening Ritual

The coordinator begins by saying "Yallah!" (Arabic: "let's go"), then checks `docs/tasks.md`, searches `docs/` for requirements, creates a TodoWrite plan, and starts coordinating agents.
