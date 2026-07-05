---
url: https://recursive-mode.dev/introduction
date_fetched: 2026-07-05
backfilled: true
---

## Solving the problem of context rot

Long-running agent work has a common failure mode: requirements, decisions, and plans live in the conversation. Once that session ends or the context window overflows, the agent loses track of what was decided, what was implemented, and why. This is**context rot**. recursive-mode solves it by making static repository documents the source of truth for every phase. Requirements, plans, and evidence live in files that persist across sessions, contributors, and repositories. Prompts become short commands, not specifications.

## Recursion in practice

recursive-mode builds on a couple of simple principles: Each development phase produces one locked output document. Each phase uses the previous phases’ output as its input. Before exiting a phase and preceeding to next, a certain set of criteria, based on the workflow as well as previous phase docs, must be fulfilled and if not, the agent needs to iterate until it reaches exit criteria.## Docs for development and traceability

With recursive-mode, all information that is important to the development workflow is stored in durable docs. This is the folder structure of a recursive run:## Docs for humans and machines, and for fine-tuning and self-distillation

Docs are human-readable and machine-readable. They offer the best traceability out of any skill or harness in 2026. Your entire rationale for building the way you did (or what the agent decided to do) is clearly recorded and referenceable. The run docs together with the code diff in the worktrees become a rich dataset for auto-training or finetuning a model against your codebase.## Chat is CLI

Chat is used the way to should be, for commands only. You can start by using plan mode and refine the plan with the agent and then ask it to turn the plan into a new recursive run. You can also create the run folder and requirements doc yourself, then ask the agent to “implement run 01”. The main point is: keep valuable information out of chat and in docs.## An alternative to Missions

recursive-mode, previously known as rlm-workflow, is a free alternative that pre-dates Factory.ai’s Missions feature by several months. It has a stronger recursion mechanism, is free and open source, and works in any IDE, CLI, agent and with any models.## Who it’s for

- **Developers who want auditable agent runs**— every phase is recorded in repo files, not in ephemeral chat
- **Teams who need recorded evidence**— requirements and implementation outcomes are captured as durable artifacts
- **Users running long or resumable work**— runs can be paused and resumed without losing context

## Key benefits

## Context in the repo

Keep important implementation context in repository files instead of losing it in chat history.

## Auditable and resumable

Make agent work easier to audit, review, and resume — even across sessions and contributors.

## Explicit evidence

Reduce vague “done” claims by requiring explicit evidence and phase completion records.

## Structured reliability

Improve reliability through structured planning, testing, review, and closeout phases.

## Safer delegated work

Make delegated or subagent work safer by requiring the controller to verify results against real files and diffs.

## Durable lessons

Preserve project decisions and operational lessons in a reusable memory layer that future runs can build on.

## What’s included

recursive-mode ships these installable skills:| Skill | Purpose | 
|---|---|
| `recursive-mode` | Core workflow orchestration: staged phases, locked artifacts, durable memory | 
| `recursive-worktree` | Isolates implementation work in a dedicated git worktree | 
| `recursive-debugging` | Structured debugging with explicit root-cause analysis (Phase 1.5) | 
| `recursive-tdd` | Strict or pragmatic TDD with recorded RED/GREEN evidence | 
| `recursive-review-bundle` | Packages delegated code reviews into canonical, reproducible bundles | 
| `recursive-subagent` | Controls subagent handoff contracts and self-audit fallback |
