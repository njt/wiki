---
url: https://recursive-mode.dev/introduction
title: "Introduction to recursive-mode"
author: try-works
date_fetched: 2026-05-14
date_published: unknown
topics:
  - specifications-as-the-product
  - agent-coding-workflow
---

# Introduction to recursive-mode

What recursive-mode is, who it's for, and the problems it solves.

recursive-mode is an installable skill package for structured AI-assisted software development. It gives your agent a file-backed workflow for requirements, planning, implementation, testing, review, closeout, and memory — instead of leaving that entire process scattered across chat history.

## Solving the problem of context rot

Long-running agent work has a common failure mode: requirements, decisions, and plans live in the conversation. Once that session ends or the context window overflows, the agent loses track of what was decided, what was implemented, and why. This is **context rot**.

recursive-mode solves it by making static repository documents the source of truth for every phase. Requirements, plans, and evidence live in files that persist across sessions, contributors, and repositories. Prompts become short commands, not specifications.

## Recursion in practice

recursive-mode builds on a couple of simple principles: Each development phase produces one locked output document. Each phase uses the previous phases' output as its input. Before exiting a phase and preceeding to next, a certain set of criteria, based on the workflow as well as previous phase docs, must be fulfilled and if not, the agent needs to iterate until it reaches exit criteria.

recursive-mode is "recursive" because the process continuously revisits its own outputs. Each phase consumes artifacts from earlier phases. Audited phases loop through `draft → audit → repair → re-audit` until the work is genuinely ready. Closeout phases feed validated lessons back into decisions, state, and memory so future runs start from better context. Later work always refers back to earlier work.

## Docs for development and traceability

With recursive-mode, all information that is important to the development workflow is stored in durable docs. This is the folder structure of a recursive run:

```
.recursive/
├── /memory/              # Structured memory bank
├── RECURSIVE.md          # Canonical workflow spec
├── STATE.md              # Current repository state
├── DECISIONS.md          # Decisions ledger
├── run/00-my-first-requirements/                  
    ├── 00-requirements.md          # User-created requirement
    ├── 01-as-is.md                 # Analysis of codebase current state
    ├── 02-to-be.md                 # Implementation plan
    ├── 03-implementation-summary.md  # What was done in practice
    ├── 04-test-summary.md        # Automated test summary
    ├── 05-manual-qa.md           # Test cases
    └── addenda/                  # Addenda docs added as needed
```

Global STATE.md and DECISIONS.md are read at the start of and updated at the end of each run.

## Docs for humans and machines, and for fine-tuning and self-distillation

Docs are human-readable and machine-readable. They offer the best traceability out of any skill or harness in 2026. Your entire rationale for building the way you did (or what the agent decided to do) is clearly recorded and referenceable.

The run docs together with the code diff in the worktrees become a rich dataset for auto-training or finetuning a model against your codebase.

## Chat is CLI

Chat is used the way to should be, for commands only. You can start by using plan mode and refine the plan with the agent and then ask it to turn the plan into a new recursive run.

You can also create the run folder and requirements doc yourself, then ask the agent to "implement run 01".

The main point is: keep valuable information out of chat and in docs.

## An alternative to Missions

recursive-mode, previously known as rlm-workflow, is a free alternative that pre-dates Factory.ai's Missions feature by several months. It has a stronger recursion mechanism, is free and open source, and works in any IDE, CLI, agent and with any models.

## Who it's for

* **Developers who want auditable agent runs** — every phase is recorded in repo files, not in ephemeral chat
* **Teams who need recorded evidence** — requirements and implementation outcomes are captured as durable artifacts
* **Users running long or resumable work** — runs can be paused and resumed without losing context

## Key benefits

**Context in the repo**: Keep important implementation context in repository files instead of losing it in chat history.

**Auditable and resumable**: Make agent work easier to audit, review, and resume — even across sessions and contributors.

**Explicit evidence**: Reduce vague "done" claims by requiring explicit evidence and phase completion records.

**Structured reliability**: Improve reliability through structured planning, testing, review, and closeout phases.

**Safer delegated work**: Make delegated or subagent work safer by requiring the controller to verify results against real files and diffs.

**Durable lessons**: Preserve project decisions and operational lessons in a reusable memory layer that future runs can build on.

## What's included

recursive-mode ships these installable skills:

| Skill                     | Purpose                                                                      |
| ------------------------- | ---------------------------------------------------------------------------- |
| `recursive-mode`          | Core workflow orchestration: staged phases, locked artifacts, durable memory |
| `recursive-worktree`      | Isolates implementation work in a dedicated git worktree                     |
| `recursive-debugging`     | Structured debugging with explicit root-cause analysis (Phase 1.5)           |
| `recursive-tdd`           | Strict or pragmatic TDD with recorded RED/GREEN evidence                     |
| `recursive-review-bundle` | Packages delegated code reviews into canonical, reproducible bundles         |
| `recursive-subagent`      | Controls subagent handoff contracts and self-audit fallback                  |

Install the full package:

```bash
npx skills add try-works/recursive-mode --skill '*' --full-depth
```

Or install a single subskill:

```bash
npx skills add try-works/recursive-mode --skill recursive-tdd --full-depth
```
