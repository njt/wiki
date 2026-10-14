# Production-Grade LLMs and Agents — A Field Guide

Stack Overflow's series opener argues that the gap between a demo agent and a production one is closed not by bigger models but by the boring machinery around them, and gives a seven-level maturity model (Notebook → Determinism → Evaluation → Confidence → Safety & governance → Operability → Production-grade build) with a self-assessment and a map to 25 deep-dive posts.

---

## The precis

The one-paragraph thesis is unusually crisp for a series index: LLMs are probabilistic, production demands guarantees, and since you can't make the model deterministic, you **shrink the model's job to the smallest decision that needs judgment** and wrap that decision in deterministic machinery you can test, gate, audit, and observe. The model is "one contained component in an otherwise ordinary, well-engineered system." The rest of the piece is a maturity model — seven levels, each assuming the one below, each with a diagnostic "tell" — plus a ten-item self-assessment and a table of contents for the deep-dive series.

## Key quotes

> You get guarantees not by making the model deterministic (you can't) but by **shrinking the model's job to the smallest decision that needs judgment**, then wrapping that decision in deterministic machinery you *can* test, gate, audit, and observe.

This is containment-as-architecture, and it's the right framing precisely because it refuses the two popular failure modes: pretending prompting discipline is engineering, and pretending the model can be sprinkled throughout a system. It dovetails with the Behavior-Tree/harness school ([[Dmitry Sotnikov's LLM Workflow]], [[I Stopped Coding and Started Architecting Agents (And Why You Should Too)]]) — model as leaf node, determinism as the trunk.

> Most teams are strong at 0 and wish for 5 — but determinism comes before evals, evals before trusting confidence, confidence before automating, safety before scaling, observability before sleeping at night.

The ordering claim is the most falsifiable and most useful thing here. Teams routinely try to buy level 5 (observability tooling) while living at level 1 (agents that can write business state and whose behavior depends on the path they wandered). The model says that's wasted spend, and the diagnosis matches what platform teams actually report ([[A Platform Engineering Playbook for Production LLMs]]).

> You don't reach production by trusting the model more; you reach it by *containing* it more.

A clean inversion of the industry's default marketing frame. Note the maturity model's level 6 is telling: production-grade isn't just runtime properties but "the *build itself* is disciplined" — an OS for coding agents and a decision log, i.e. the same insight the agentic-coding wiki has been converging on from the practitioner side.

## Key themes

- #concept — The maturity ladder as ordered dependency: determinism → evals → calibrated confidence → safety → operability; skipping rungs is the canonical anti-pattern
- #pattern — Propose/apply separation: "an agent can only *propose*; a separate component applies changes after approval" as the level-1 gate
- #pattern — Fail-closed layered guardrails, append-only audit ledger, PII redaction at the boundary — defense-in-depth ported from security engineering to LLM systems
- #tool — Kill switch "in seconds without a deploy" as the operational test of whether you actually have control

## Analysis

This is a map piece, not an argument piece — its value is the ordering and the diagnostic tells, and those are good. The self-assessment is the sharpest artifact: ten binary questions where "fewer than half ticked → you're earlier than the demo suggests" is exactly the mirror most teams need. The weakness is the usual one for maturity models: it's a checklist of *properties* with little to say about the economics of getting there, and level 6's pivot to "the build itself" quietly imports an entire agentic-coding practice without argument. The piece also assumes a greenfield-shaped system ("fixed graph per capability"), which brownfield reality complicates. Still, as an index into 25 deep-dives it's the best single skeleton for the production-LLM topic this wiki has ingested — it gives [[A Platform Engineering Playbook for Production LLMs]] a place to live, and its levels 1–2 map almost one-to-one onto the loop-engineering literature.

## Relations

- Strengthens [[A Platform Engineering Playbook for Production LLMs]]: that field report's gateway/evals/attribution/cost-contents are exactly levels 2 and 5 of this maturity model, arrived at empirically — the ladder generalises the case study.
- Nuances [[Feedback Loop is All You Need]]: the evals-as-CI-gate and drift-detection levels give the feedback-loop thesis a concrete ordering and a precondition (determinism first) that pure loop advocacy tends to skip.
- Complicates [[Own the Outer Loop]]: Osmani locates human judgment in the outer loop; this model locates it in the composed-confidence/auto-vs-human threshold at level 3 — a narrower, more mechanised role for the human than Osmani's accountability framing suggests.

---
*Sources: [[raw/production-grade-llms-and-agents-a-field-guide]], [[summary/production-grade-llms-and-agents-a-field-guide]]*
*Last updated: 2026-10-14*
