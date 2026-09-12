---
url: https://paperclipai.net/
title: "Paperclip — Open-Source Orchestration for Zero-Human Companies"
author: Paperclip
date_fetched: 2026-09-11
date_published: unknown
topics:
  - agent-orchestration
---

# Source Analysis: Paperclip (paperclipai.net)

## What It Is

Paperclip is an open-source (MIT), self-hosted orchestration layer for running "zero-human companies." The pitch: hire AI employees into an org chart — CEO, CTO, engineers, designers, marketers — set a company goal ("Build the #1 AI note-taking app to $1M MRR"), approve the strategy, and let heartbeats keep the lights on. You are the board of directors; the agents do the work.

## The Model

Paperclip is explicitly *not* an agent framework. It does not build agents, write their prompts, or pick their models — it models the **company** they work in. Its primitives are org charts (hierarchies, roles, reporting lines), monthly per-agent budgets with hard stops, goal-to-task context inheritance, a ticket system with an immutable audit log, and board-level governance (approve hires, override strategy, pause/terminate any agent).

**Bring your own agent.** Any runtime that can receive a heartbeat is hireable — Claude Code sessions, OpenClaw bots, Codex, Cursor, Python scripts, shell commands, HTTP webhooks. Adapters connect Paperclip to whatever executes them.

**Heartbeats.** Agents wake on a schedule, check work, and act; delegation flows up and down the org chart; ticket assignments and @-mentions also wake agents.

**Cost control.** Every agent has a monthly budget. At 100% utilization the agent auto-pauses and blocks new tasks (soft warning at 80%); the board can override. Budgets and task checkout are enforced atomically, so no double-work and no runaway spend.

**Governance.** Agents can't hire agents without approval; the CEO can't execute an unreviewed strategy; config changes are revisioned and rollbackable. "Autonomy is a privilege you grant, not a default."

**Multi-company.** One deployment runs dozens of companies with complete data isolation and separate audit trails.

## The Hard Details It Claims to Get Right

Atomic execution (checkout + budget), persistent agent state across heartbeats, runtime skill injection via a `SKILLS.md`, governance with rollback, goal-aware execution (tasks carry full goal ancestry), portable company templates with secret scrubbing, and company-scoped entity isolation.

## Positioning

Deliberately narrows itself by negation: not a chatbot, not an agent framework, not a workflow builder, not a prompt manager, not a single-agent tool. "If you have one agent, you probably don't need Paperclip. If you have twenty — you definitely do." Local setup is a single Node.js process with an embedded Postgres; interactive setup handles database, auth, and the first company, with no Paperclip account required.
