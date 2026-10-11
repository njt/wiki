# Autonomous Delivery (Evan Boyle)

Evan Boyle's open-source Agent Skill (v2.1.0, MIT, a gist of SKILL.md + README) that encodes an *adaptive* orchestrator-worker methodology as installable instructions: a strong outer coordinator delegates outcomes to workers or local orchestrators, then repeatedly reflects on and reshapes its own organization — boundaries, specialization, hierarchy, concurrency — while a strict authority/evidence/idempotency floor stays fixed. Its thesis: the organization is a variable to optimize, not a pipeline to run.

---

## Key Quotes

> "Start with the organization the task needs. Grow, simplify, or reshape it as evidence arrives. … More agents are not the goal, and fewer are not inherently better either."

The anti-dogma stance that distinguishes this from fixed multi-agent pipelines. It even says a small bug should be judged "by the regression evidence, not by whether orchestration occurred" — staying small is a valid optimization.

> "Increasing all three [capability, reasoning effort, context size] for every call is not a strategy."

The model-selection table routes work by leverage: strongest reasoning for design decisions, concurrency invariants, hard bugs and unresolved ambiguity; fast capable workers for bounded implementation with clear contracts. Explicit escalation rules: hand the *failing case plus attempted explanations* to the stronger model, return its conclusion to the existing owner rather than restarting the workstream.

> "Use hierarchy to contain complexity, not to reproduce an org chart."

Layers are added only when they reduce the coordinator's cognitive burden and collapsed when "the extra handoffs cost more than they save." A local orchestrator receives "responsibility for its outcome, not permission to multiply agents without purpose," and may narrow but never widen its delegated envelope.

> "A timed-out external write has an unknown outcome, not necessarily a failed one."

The safety floor: skills grant no permissions, busy is not proof of progress, silence is not proof of a stall, and uncertain effects must be reconciled by receipt and live state before retrying — distributed-systems discipline applied to agent fleets.

> "Execution feedback asks, 'Is this result right?' Workflow reflection also asks, 'Is this organization helping us get better results faster?'"

The reflection loop (observe friction → hypothesize cause → change one variable → compare quality *and* throughput → keep or undo) treats workflow itself as an experiment subject, with confounders acknowledged ("a faster easy task does not prove a better process").

## Key Themes

#orchestration #multi-agent #skills #delegation #model-selection #patterns

## Analysis

This is the most disciplined articulation of adaptive multi-agent organization I've seen as a *shippable artifact* rather than an essay. Its strength is refusing every tempting extreme: no fixed pipeline, no agent-count maximization, no permanent model hierarchy, no reflexive fan-out. The recurring-work section is the part most other orchestrator-writeups skip entirely — the same-session-wakeup vs. fresh-session-job distinction, watermark-based triage ("do not create a task to justify the wakeup"), and coalescing wakeups behind active work are real operational wisdom, and the scheduled-wakeup instruction template is directly reusable.

Weaknesses: it is long and abstract for a skill — much of it is methodology prose that a mid-tier worker may not reliably apply, which the author half-acknowledges by saying "without session orchestration, apply the reasoning in a single agent." The reflection metrics are named but not operationalized (no thresholds, no instrumentation contract). And it depends heavily on host primitives (session creation, completion events, scheduled wakeups) that most Agent Skills clients don't actually expose — the portability claim is aspirational. Still, as a normative spec for what a *good* orchestrator does, it sets a high bar.

## Relations

The adaptive-reshape loop strengthens [[Dynamic Workflows in Claude Code]] — Anthropic's generated-harness approach shares the "organization as variable" insight but offers composable fixed patterns where Boyle insists the pattern itself must be revisited mid-run.

It nuances [[The Advisor Strategy]]: Anthropic's advisor-executor is a *fixed* two-model pairing, exactly the "hard-coded permanent model pairing" Boyle warns against; his model-selection table generalizes it into a per-call routing decision.

It complements [[108 PRs in Eight Days — Accidentally Discovering Loop Engineering]] — Brittany Ellich discovered loop/worker separation empirically; Boyle ships the codified version, including the same single-writer ownership and checkpoint discipline.

It complicates [[Orchestrator - Worker Skill]] — that skill merges orchestrator and worker into one file for simplicity; Boyle's answer is the opposite: keep roles separate but make the *shape* of the hierarchy itself mutable.

---
*Sources: [[raw/8c1a97f682d92dab8c09d0e1ea73f4a0]], [[summary/8c1a97f682d92dab8c09d0e1ea73f4a0]]*
*Last updated: 2026-10-11*
