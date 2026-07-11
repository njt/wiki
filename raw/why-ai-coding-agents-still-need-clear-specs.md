---
url: https://www.oreilly.com/radar/why-ai-coding-agents-still-need-clear-specs/
title: Why AI Coding Agents Still Need Clear Specs
author: Markus Eisele
date_fetched: 2026-07-11
date_published: 2026-07-08
site: O'Reilly Radar
---

# Why AI Coding Agents Still Need Clear Specs

The article opens by identifying a prevalent mindset in the developer community: that AI agents are smart enough to operate without heavy upfront specification. The author argues this is wrong, not because agents lack capability, but because "the accounting is off." He asserts that minimal specification doesn't eliminate cost—it defers and fragments it, making it harder to detect.

## Two poles, two hidden costs

The article contrasts minimal specification (near-zero upfront human effort, but downstream costs from correction loops, token expenses, and human re-engagement) with full formal specification like TDD/BDD (visible upfront human effort, but automated downstream verification that doesn't fatigue). The key trade-off involves *when* you pay and *in what currency*. Minimal spec "front-loads token cost and back-loads human judgment," while heavy spec "front-loads human effort and back-loads almost nothing."

Total cost across both approaches traces a U-shaped curve against specification completeness. The minimum falls around well-structured acceptance criteria or BDD scenarios—not at zero spec, and not at a 40-page formal document. Multi-agent work pushes that minimum further right because drift compounds across handoffs.

## The old problem was always the spec

The author states the real challenge has always been specification—agreeing what should exist, what should never happen, which trade-offs matter, and what "done" means. Agents don't remove that problem; they make it more visible. For decades, the spec problem was hidden inside meetings, backlogs, code reviews, and QA cycles. Agents reduce the friction of producing code, which means missing pieces surface later since the system "can now produce a plausible implementation before anyone has really decided what the implementation is supposed to mean."

In the old world, vague requirements ran into human slowness. In the agent world, "vague requirements run into machine speed."

## But writing the spec is only half the problem

The article notes that a spec must be validated before being handed to an agent. A spec can fail in invisible ways—internal inconsistency, incompleteness (covering the happy path but not edge cases like a 429 API response), untestability, or being precisely what was written but not what was meant. An agent executing faithfully against a flawed spec produces something difficult to debug; the correction loop grows more expensive because you must unwind both code and reasoning.

Spec validation is a distinct cost category between "write spec" and "run agent," requiring human time, agent time, or both—and it isn't zero.

## How agents can write specs

A third strategy exists: using agents to write and validate the spec, then using implementation agents to execute against it. A spec-drafting agent produces a first version from rough intent. A spec-validation agent stress-tests that draft for consistency, completeness, and testability. A test-writing agent translates surviving claims into executable checks. Humans review the result faster than writing from scratch.

The best prompt is not "write me a spec." Instead, the author suggests something closer to:

> Draft the smallest spec that would let another agent implement this safely. Include assumptions, nongoals, acceptance criteria, edge cases, observable outcomes, and open questions.

Then a different agent attacks that output, finding contradictions, ambiguous terms, hidden dependencies, untestable claims, and places where "an implementation could pass the written criteria while still violating the intent."

## How BDD partially solves this

Behavior-driven development, when done well, collapses spec writing and validation into the same artifact. A Gherkin scenario is simultaneously intent description and executable test. Running the spec against a skeleton implementation immediately reveals whether the description produces coherent behavior. Making the spec executable forces validation that prose doesn't—some ambiguity must be resolved before the scenario can run.

"BDD earns its keep when it moves judgment out of repeated human review and into an executable oracle." However, Gherkin can still be written badly, and ambiguity can live in semantics rather than syntax.

## Multi-agent pipelines break everything

For a single agent on a well-bounded task, underspecification is recoverable. Multi-agent pipelines are different: when Agent A's output becomes Agent B's input, any interpretive drift from A compounds into B's execution. B "works hard and confidently on the wrong foundation." Errors get amplified and obscured through multiple layers.

This pushes the breakeven point decisively toward specification. In multi-agent systems, a spec is "a coordination contract between agents." The less precise that contract, the more each agent's interpretive freedom introduces variance. The author argues for strongly typed interfaces between agents rather than loose conversational handoffs.

## What survives from methodology

The article notes that agile as theater—standups, estimation rituals, ticket ceremonies—is in trouble. Agents don't need those, and honestly, humans didn't either.

What survives: agile as a feedback philosophy, short cycles, working software over abstract progress, customer collaboration, and plans bending when reality speaks. XP survives: test-first thinking, human-agent pairing, continuous integration, refactoring, and small releases. "Large invisible deltas are where both humans and agents lose the plot."

The question is not which methodology "wins" but which practices "still reduce the cost of discovering that the spec was wrong."

## Where to actually invest

The optimal investment point is task-dependent, and the honest calculation includes spec validation as real cost.

- **Single agent, small bounded task:** structured intent with goal, examples, nongoals, and a few acceptance criteria. BDD may be overkill; zero spec is "still lazy accounting."

- **Deterministic, well-understood work** (API integrations, CRUD, data transformations): more specification pays off faster; skimping defers rework.

- **Exploratory or creative work:** over-specification constrains valuable flexibility; the breakeven sits left, but boundaries around exploration are still needed.

- **Multi-agent systems:** the sweet spot shifts right again. "The handoff is the product." Every agent boundary needs a contract with schema, invariants, allowed ambiguity, and validation checks. Otherwise "you're not orchestrating agents. You're compounding interpretations."

The conclusion: "Validate your spec." Whether through human review, agent stress-testing, or executable formats like BDD, skipping validation means paying later at higher interest with worse diagnostics. "The agents are getting better. The accounting problem is still ours."
