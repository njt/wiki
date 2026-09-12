---
url: https://www.oreilly.com/radar/why-ai-coding-agents-still-need-clear-specs/
title: "Why AI Coding Agents Still Need Clear Specs"
author: Markus Eisele
date_fetched: 2026-07-11
date_published: 2026-07-08
topics:
  - agent-orchestration
  - specifications-as-the-product
---

# Why AI Coding Agents Still Need Clear Specs — Markus Eisele

The article challenges the notion that AI coding agents are smart enough to operate without heavy upfront specification. The core argument is an accounting one: minimal specification doesn't eliminate cost — it defers and fragments it into correction loops, token expenses, and repeated human re-engagement.

Two poles anchor the trade-off. Minimal spec has near-zero upfront human effort but high downstream costs as vague requirements meet machine speed. Full formal spec (TDD/BDD) front-loads human effort but back-loads almost nothing. Total cost traces a U-shaped curve against spec completeness; the minimum sits around well-structured acceptance criteria, not at zero and not at a 40-page formal document.

The real challenge has always been specification — agreeing what should exist, what must never happen, and what "done" means. Agents don't remove that problem; they make it more visible by producing plausible implementations before anyone has decided what the implementation is supposed to mean.

Spec validation is a distinct cost category between writing the spec and running the agent. A spec can fail in invisible ways — internal inconsistency, missing edge cases, untestability, or being precisely what was written but not what was meant. An agent faithfully executing a flawed spec produces output that is expensive to debug because you must unwind both code and reasoning.

A third strategy uses agents themselves: a spec-drafting agent produces a first version from rough intent, a spec-validation agent stress-tests it, a test-writing agent translates claims into executable checks, and humans review the result faster than writing from scratch. BDD partially solves validation by collapsing spec and test into the same artifact — a Gherkin scenario is simultaneously intent description and executable oracle.

Multi-agent pipelines make underspecification far more dangerous. When Agent A's output becomes Agent B's input, interpretive drift compounds. Each agent boundary needs a contract with schema, invariants, and validation checks — otherwise you're "compounding interpretations, not orchestrating agents."

The optimal investment point is task-dependent: single-agent bounded tasks need structured intent with acceptance criteria; deterministic work repays specification quickly; exploratory work should lean left but still needs boundaries; multi-agent systems push the sweet spot decisively right. Agile-as-theater (standups, estimation rituals) is in trouble, but agile-as-feedback-philosophy and XP practices survive.

The conclusion: validate your spec. Whether through human review, agent stress-testing, or executable formats, skipping validation means paying later at higher interest with worse diagnostics. "The agents are getting better. The accounting problem is still ours."
