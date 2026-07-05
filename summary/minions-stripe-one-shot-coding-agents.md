---
url: https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents
title: "Minions: Stripe's One-Shot, End-to-End Coding Agents"
author: Alistair Gray
date_fetched: 2026-05-14
date_published: 2026-02-09
---

# Minions: Stripe's One-Shot, End-to-End Coding Agents

**Author:** Alistair Gray
**Published:** February 9, 2026
**Type:** Blog (Part 1 of a miniseries)

## Summary

Stripe built a homegrown system of unattended coding agents called "Minions" that handle over a thousand pull requests per week. Humans review the code, but Minions write it entirely from scratch, start to finish, without any human intervention during the process.

## Key Themes & Concepts

- **Unattended, one-shot coding agents** — Minions operate without human interaction between initiation and PR creation
- **Parallelization of developer attention** — Engineers can spin up multiple Minions simultaneously, especially valuable during on-call rotations
- **"Shift feedback left"** — The principle of catching issues as early as possible in the development cycle
- **Stripe-specific constraints** — Hundreds of millions of lines of code, Ruby + Sorbet typing, vast homegrown libraries unique to Stripe, handling over $1 trillion in payment volume
- **"If it's good for humans, it's good for LLMs"** — Minions use the same internal developer tooling as human engineers
- **Diminishing marginal returns** — At most two rounds of CI per Minion run; the system favors speed and pragmatism
- **Agent + deterministic hybrid** — The core loop mixes LLM creativity with deterministic steps for git, linters, and testing

## Tools & Infrastructure Mentioned

| Tool | Role |
|------|------|
| **Goose** (Block's coding agent) | Forked and customized as the core agent loop |
| **MCP (Model Context Protocol)** | Common language for LLM function calling across all Stripe agents |
| **Toolshed** | Central internal MCP server hosting 400+ tools |
| **Devboxes** | Isolated, pre-warmed developer environments spun up in ~10 seconds |
| **Sourcegraph** | Code intelligence and search used for context gathering |
| **Claude, Cursor, Claude Code** | Human-operated coding tools used alongside Minions |
| **Sorbet** | Ruby type checker used throughout Stripe's backend |

## How Minions Work (Chronological Flow)

1. **Initiation** — Triggered via Slack (most common), CLI, web interface, or from internal apps (docs platform, feature flag platform, ticketing UI)
2. **Environment setup** — A devbox spins up in ~10 seconds, pre-warmed with Stripe code and services, isolated from production
3. **Context hydration** — MCP tools are run deterministically over likely-looking links *before* the run starts; the agent can also access Slack threads and any linked context
4. **Agent loop** — Runs on a forked version of Block's goose, interleaved with deterministic code for git operations, linters, and testing
5. **Rule file consumption** — Minions read the same coding agent rule files as Cursor and Claude Code, with conditional rules applied per subdirectory
6. **Testing & iteration** — Local executables run lints on each push (<5 seconds); CI runs selectively from 3+ million tests; autofixes applied automatically; at most 2 rounds of CI
7. **Output** — A branch is created, pushed to CI, and a PR is prepared following Stripe's PR template

## Entry Points

Engineers most frequently start Minions from Slack by tagging the Slack app in a thread. Additional entry points include the CLI, a web UI, and buttons embedded in internal applications (e.g., flaky test tickets automatically create tickets with a "fix with Minion" button).

## Notable Quotes

> "a thousand pull requests merged each week at Stripe are completely minion-produced"

> "one of our most constrained resources is developer attention"

> "unattended agents allow for parallelization of tasks"

> "we only have at most two rounds of CI"

> "if it's good for humans, it's good for LLMs, too"

> "shift feedback left"

> "there are diminishing marginal returns for an LLM to run many rounds of a full CI loop"

> "devboxes are pre-warmed so one can be spun up in 10 seconds"

> "our North Star is a pull request produced without any human code"

> "a minion run that's not entirely correct is often still an excellent starting point"

> "Vibe coding a prototype from scratch is fundamentally different from contributing code to Stripe's codebase"

## What's Next

Part 2 will dive deeper into how Minions were implemented.
