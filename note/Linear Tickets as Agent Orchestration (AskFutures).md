# Linear Tickets as Agent Orchestration (AskFutures)

AskFutures repurposes Linear tickets from a "who's working on what" tracker into three jobs for the agent era: recording decisions, orchestrating parallel agents through a parent-child dependency graph, and batching integration into one pull request per parent. A lightweight text-file decision index makes the recorded history findable again months later.

---

## What it argues

The essay's core move is inverting the ticket's traditional question. Ticketing was built for visibility of human activity; once agents do the implementation, that question is mostly answered by the queue itself. What's scarce instead is (a) durable *why* — "code remembers what changed. Tickets remember why" — and (b) coordination structure for many agents working at once.

Concretely:

- **Decisions, not activity.** Tickets record decisions and meaningful deliverables, not every internal agent step. Otherwise agent-session context evaporates when the session ends.
- **Ticket tree as execution graph.** Parent ticket → child tickets with dependencies. Independent children run in parallel; blocked children wait. The unit-of-work redefinition is the sharpest idea here: a good child ticket is *not* a small task but one an agent can complete "with minimal coordination and minimal conflict with other agents."
- **One parent PR.** When merge gates are slow, one-PR-per-child pays the full validation cost repeatedly. Grouping children under one parent PR amortises the gate while keeping parallel development.
- **Decision index.** Recording is easy; retrieval is hard. A small text file holds searchable descriptions and pointers — not copies — and agents follow progressive disclosure: map → Linear ticket → code/PR only if needed. Descriptions are written for discovery ("search layout, pagination, chart height, mobile navigation") rather than taxonomy.

## Key quotes

> Code remembers what changed. Tickets remember why.

The aphorism that carries the whole piece. It reframes tickets as the memory layer between disposable agent sessions and the permanent record — which is exactly the gap left when an agent's context dies at session end.

> A good child ticket isn't necessarily a small task. It's a piece of work an agent can complete with minimal coordination and minimal conflict with other agents.

This quietly overturns the standard decomposition heuristic (small, independent tasks optimised for *human* context-switching). Decomposition for agents optimises for conflict-freedom and self-containment, which can mean larger tickets, not smaller ones.

> We don't use tickets to count how busy agents are. We use them to keep parallel work coherent.

A pointed jab at activity-metric ticketing, and the same anti-metric stance found across the agentic-workflow literature.

## Key themes

#concept #pattern #tool

## Critical take

This is a compact, practitioner-grounded statement of what ticketing becomes when agents outnumber humans on the queue — and the decision-index section is the most practically valuable part, because it addresses the problem everyone else skips: recorded decisions are worthless if agents can't *find* them. The progressive-disclosure ladder (map → ticket → code) is a cheap, tool-agnostic pattern.

The weak joints: the one-parent-PR strategy trades review granularity for gate cost, and the piece never confronts what happens to review when five children land in a single diff — a real tension with the [[AI Code Review]] literature's complaint that oversized agent diffs are already unreviewable. And "decisions live in tickets" risks duplicating what ADRs already do; the essay never says how the two layers relate. It's also notable how tool-coupled the setup is to Linear's particular parent-child model, which limits transferability.

## Relations

- [[Nobody Could Have Written the Ticket]] complicates this from the other direction: James Randall argues the ticket itself — the upstream artifact this whole system depends on — is frequently unwritable in advance, so a decision-record built on tickets inherits that fragility.
- [[Lovelace]] is the file-based cousin: tickets, ADRs and agent session records as Markdown in `.lovelace/` rather than Linear. AskFutures is the hosted-SaaS version of the same "durable artifacts beside agents" thesis.
- [[Trying the Software Factory Pattern (Larson)]] shows the Linear-native side already emerging in practice — a `/linear-project-loop` skill auditing and working Linear projects — strengthening AskFutures' claim that Linear is becoming an agent control plane.
- [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] supplies the failure mode this design prevents: decisions silently resolved inside an agent session leave no record. AskFutures' "tickets remember why" is one concrete tripwire against exactly that debt.

---
*Sources: [[raw/reinvent-software-tickets-for-the-ai-era]], [[summary/reinvent-software-tickets-for-the-ai-era]]*
*Last updated: 2026-11-17*
