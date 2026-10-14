# The Flight Recorder — Replaying What an AI Agent Did on My SQL Server

The AgentDBA author shows how to make an AI agent auditable on a production database: every decision in a run is written as one row in the agent's own database — who made it (code vs. model), the stated reason, the action, the latency — so that a complete 11.2-second health check can be replayed from two SELECT queries. Trust comes not from the model's word but from the record.

---

## The Worked Example

One full health check, 11.2 seconds, replayed row by row:

| Step | At (ms) | Decided by | Action |
|---|---|---|---|
| 1 | 22 | code | db_status |
| 2 | 67 | code | error_log |
| 3 | 2,698 | LLM | backup_failures |
| 4 | 5,253 | LLM | failed_jobs |
| 5 | 8,220 | LLM | log_file_health |
| 6 | 11,227 | LLM | CONCLUDE (CRITICAL) |

Three details carry the design:

- **Two columns matter before any detail**: how many of the six decisions the model made, and who closed the run. "An agent's record should always separate what the AI chose from what code chose for it."
- **The first two steps were not the model's call.** Every health check opens with database status and the error log, executed by code. Had either returned CRITICAL, the agent escalates with no model involved. Only then does the model choose, "inside a fixed boundary": three checks, each allowed once, then one verdict, with a stated reason for every action.
- **Evidence behind the verdict.** Each tool step carries an `audit_log_id` pointing to one stored row holding the raw resultset SQL Server returned *and* the compact summary the model was shown. The model never sees raw rows — ordinary code reduces each resultset to a short structured summary — but a reviewer can check the conclusion against the data the tool actually read.

## The Latency Split

The five SQL checks took 413 ms combined. Nearly all the other 10.8 seconds was the model deciding what to do next: four decisions at roughly 2.5–3 seconds each. The agent is not slow because of the database; it is slow because of deliberation — and a DBA reading the record can see exactly where the time went.

## Key Quotes

> "An agent's record should always separate what the AI chose from what code chose for it."

The whole audit model in one sentence. Blurred attribution is what makes agent logs unusable: if you can't tell whether a deterministic gate or the model decided a step, you can neither debug the harness nor assign trust. This is the runtime version of the determinism-first principle in [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] — and here it's not just a design choice, it's a column in the log.

> "I did not want to ask a DBA to take an AI agent's word for anything on a production server."

The motivation is institutional, not technical. A DBA's job is defensibility; an agent that can't show its work can't be allowed near production, whatever its accuracy. The recorder is the answer to that org reality, in the same spirit as the audit-trail commitments in [[Building Shippy — Agent Architecture for High-Stakes Domains]].

> "The recorder is my answer: one row per decision, kept in your own database, readable with a SELECT."

Note the deliberate mediocrity of the medium: no observability vendor, no trace format, just the database the DBA already lives in. Accessibility *is* the feature — an audit trail nobody can query is a trust mechanism in name only.

## Key Themes

#concept — code-vs-LLM decision attribution as a first-class logged field
#pattern — the flight recorder: one row per decision with reason, action, latency
#tool — AgentDBA, a SQL Server health-check agent with a built-in audit log
#concept — raw-resultset vs. model-visible-summary separation for post-hoc audit

## Critical Analysis

**The strongest idea is attribution, not logging.** Plenty of agent frameworks emit traces; few log *who decided* as a queryable column. The code-gate/LLM-boundary split here is genuinely good harness design: the two cheapest and most consequential checks (is the database up, what's in the error log) are deterministic, and the model only operates "inside a fixed boundary" with a per-action reason requirement. That reason requirement is the same move Pinal Dave found transformative in schema mapping — forcing justification turns a useless confidence signal into a triageable artifact ([[AI-Assisted Database Work — The Machine Reads, The Human Decides]]).

**The raw-vs-summary split is quietly the deepest part.** Storing both the raw resultset and the compacted summary is what makes the audit *checkable* rather than merely narratable: you can ask not just "why did it conclude CRITICAL?" but "was the conclusion consistent with data the tool actually read?" This is context engineering ([[Agent Memory and Context]]) married to accountability — the compaction that saves tokens is also the thing that creates an evidence gap, and the design closes it.

**It's marketing, and the record is thin.** This is a product blog post (AgentDBA has "a free evaluation at agentdba.ai"), and it replays a lab run on a local server. There's no discussion of failure cases, log volume, retention, or what happens when the model's stated reason is post-hoc rationalisation rather than the actual cause. The flight recorder tells you what the agent *said* it was doing; it doesn't guarantee the reason field is honest. That's the gap between an audit log and an explanation, and the post doesn't touch it.

**The latency observation deserves more than a paragraph.** 413 ms of SQL against 10.8 seconds of deliberation is the general shape of agentic systems — tool calls are cheap, decisions are expensive. The interesting question the post raises but doesn't pursue: with deterministic gates and a bounded check budget, at what point does the fixed scaffolding do so much of the work that the model is reduced to choosing an order and writing prose?

## How This Fits the Wiki

- This is the production instantiation of the Tracer pattern in [[Building an Advanced Agentic Harness]] — same flight-recorder idea (per-step identity, action, latency, verdict), but stored in the operator's own database and designed for a DBA's SELECT rather than a developer's matplotlib plot. It nuances that page by adding the code-vs-LLM attribution column, which the tutorial tracer doesn't surface.
- It strengthens [[Building Shippy — Agent Architecture for High-Stakes Domains]]: Shippy isolated sessions and evaluated against live data; AgentDBA adds the complementary high-stakes-domain primitive — the persisted decision log that lets an outside auditor reconstruct any run.
- It complicates [[AI Handles Incidents, Engineers Lose Touch with Their Systems]]: Kalache warns that automated incident handling atrophies human intuition; the flight recorder is a partial rebuttal, since the DBA replays the reasoning end-to-end rather than accepting a verdict — though reading logs is not the same practice as handling the incident.
- It extends [[AI-Assisted Database Work — The Machine Reads, The Human Decides]] from AI *reading* databases to AI *acting* on them, with the same keystone: the human verifies, but here verification is made mechanical by the stored raw-vs-summary pairing.

---
*Sources: [[raw/the-flight-recorder-replaying-what-an-ai-agent-did-on-my-sql-server]], [[summary/the-flight-recorder-replaying-what-an-ai-agent-did-on-my-sql-server]]*
*Last updated: 2026-10-14*
