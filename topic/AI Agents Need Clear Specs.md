# AI Agents Need Clear Specs

Markus Eisele's sharp economic analysis of why "agents are smart enough to figure it out" is wrong — not because agents lack capability, but because the accounting is off. Minimal specification doesn't eliminate costs; it defers and fragments them, shifting payment from upfront human effort to downstream correction loops, token burn, and debugging marathons. The cost curve is U-shaped: the minimum sits at well-structured acceptance criteria or BDD scenarios, not at zero spec and not at 40-page formal documents. Multi-agent pipelines push that minimum further right because interpretive drift compounds across handoffs.

---

## Key Quotes

> "The accounting is off."

The thesis in four words. Every argument for skipping specs assumes costs disappear when deferred. They don't — they move to harder-to-see places with worse diagnostics. This is the same structural insight behind [[Discovery Debt]]: invisible costs compound, and "speed feels like signal. It isn't."

> "In the old world, vague requirements ran into human slowness. In the agent world, vague requirements run into machine speed."

The cleanest articulation of what's actually new. Spec weakness was always expensive, but human bottlenecks capped the damage rate. Agents remove that cap — you can now produce a plausible implementation before anyone has decided what the implementation is supposed to mean. The speed of generation amplifies the cost of unclear intent. This is the [[Compound Engineering]] problem applied to requirements: cheap code production makes expensive decisions the binding constraint.

> "A spec is a coordination contract between agents. The less precise that contract, the more each agent's interpretive freedom introduces variance."

The multi-agent insight that distinguishes this from single-agent advice. In a pipeline, Agent A's output is Agent B's input, and any drift compounds. Strongly typed interfaces between agents — schema, invariants, validation checks — aren't bureaucratic overhead; they're the only thing preventing interpretation from cascading into hallucination. This echoes [[Agent Orchestration]]'s finding that planner/worker/judge patterns keep emerging for exactly this reason.

> "Validate your spec. Whether through human review, agent stress-testing, or executable formats like BDD, skipping validation means paying later at higher interest with worse diagnostics."

Spec validation is a distinct cost category between "write spec" and "run agent" — and it's never zero. This is what [[How to Write a Good Spec for Agents]] acknowledges but doesn't fully solve: the spec itself can be wrong, incomplete, internally inconsistent, or untestable. An agent executing faithfully against a flawed spec produces something where you must unwind both code *and* reasoning.

> "The agents are getting better. The accounting problem is still ours."

The closer. Model capability improvements don't change the economic structure — they just change the multiplier on whatever spec quality you brought in.

## Key Themes

#spec-economics #agentic-development #multi-agent #bdd #methodology #accounting

## The U-Curve of Spec Investment

Eisele's central contribution is the U-shaped total cost curve. At zero spec, you pay in token burn, correction loops, and human re-engagement. At full formal specification, you pay in upfront human effort but back-load almost nothing. The minimum is somewhere in the middle — structured acceptance criteria, BDD scenarios — not at either extreme.

This reframes the spec debate from "how much specification?" to "where on the curve does this task sit?" Single-agent bounded tasks: left of center. Multi-agent pipelines: decisively right of center. The curve isn't static — it shifts with task complexity, team structure, and the cost of being wrong.

## Spec Validation as the Hidden Cost

The article's most novel point: writing the spec is only half the problem. Validation — checking for internal inconsistency, missing edge cases, untestable claims, and intent-violating implementations — is a real cost that most spec advocacy ignores. Eisele identifies three strategies for validation:

1. **Human review** — expensive but high-judgment
2. **Agent stress-testing** — cheaper, adversarial ("find where an implementation could pass the criteria while violating the intent")
3. **Executable formats (BDD)** — collapses writing and validation into one artifact, forces ambiguity resolution

The prompt he suggests for spec-drafting agents is worth stealing: "Draft the smallest spec that would let another agent implement this safely. Include assumptions, nongoals, acceptance criteria, edge cases, observable outcomes, and open questions." Then have a *different* agent attack it.

This agent-mediated spec pipeline — draft → attack → refine → execute — is a pattern [[SDDW (Spec-Driven Development Workflow)]] operationalizes but Eisele explains the economics of.

## BDD as Executable Oracle

BDD earns its keep not through better documentation but by moving judgment out of repeated human review and into an executable oracle. A Gherkin scenario is simultaneously intent and test. Running it against a skeleton immediately reveals whether the description produces coherent behavior. This is the same insight as [[The Oracle Is the Asset]]: the test suite is the durable artifact, frameworks become transpilers, and you own the spec the compiler answers to.

Eisele is honest about BDD's limits: Gherkin can be written badly, ambiguity survives in semantics not syntax, and BDD is overkill for small bounded tasks. The value is proportional to the cost of being wrong.

## Multi-Agent Systems as the Spec Stress-Test

Single-agent underspecification is recoverable with iteration. Multi-agent underspecification compounds. When Agent A's interpretive drift becomes Agent B's foundation, errors amplify through layers. This is why [[Fable Open-Sourced NanoClaw's PR Factory]] succeeded where looser orchestrations fail: the customization guidelines functioned as the coordination contract Eisele describes.

Eisele's prescription for multi-agent handoffs — schema, invariants, allowed ambiguity, validation checks — reads like an API contract between services. Which is exactly the point: multi-agent systems *are* distributed systems, and [[Distributed Systems]] teaches us that loose contracts at boundaries create the hardest bugs.

## What Survives from Methodology

The section on agile and XP is brief but precise. Agile-as-theater (standups, estimation rituals, ticket ceremonies) is dead — agents don't need it and humans didn't either. Agile-as-feedback-philosophy survives. XP's core practices — test-first, pairing (now human-agent), CI, refactoring, small releases — survive because they "reduce the cost of discovering that the spec was wrong." This aligns with [[Lean Software Production]]'s argument that methodology becomes existential, not optional, in the agentic era.

The question isn't which methodology wins but which practices still reduce the cost of being wrong. That's the right framing and it makes Eisele's conclusion inevitable: anything that catches spec errors earlier is worth more now than it was before agents.

## Critical Analysis

This is the clearest economic framing of the spec debate yet. The U-curve model is simple enough to be useful and nuanced enough to be correct — it accounts for task type, team structure, and the hidden cost of validation. The spec-validation-as-distinct-cost insight is genuinely new and addresses the gap [[Specifications as the Product]] leaves open: tools exist to test code against specs, but almost nothing tests specs against themselves.

The weakness is that Eisele doesn't reckon with the organizational incentives that make underspecification attractive. The costs of deferred specification — correction loops, token burn, debugging marathons — are borne by the team doing the work. The costs of upfront specification — visible time spent "not coding" — are borne by the person writing the spec, and they look idle to management. [[Discovery Debt]] names this dynamic ("invisible work loses, every time") but Eisele's accounting framework needs that organizational layer to be actionable.

The BDD section is the right level of enthusiasm — it's useful, not magic, and the "Gherkin can be bad too" honesty matters. The agent-mediated spec pipeline (draft → attack → refine) is where the real leverage is: it applies the same cost-reduction logic to specification that agents already apply to implementation. The best prompt in the article — "draft the smallest spec that would let another agent implement this safely" — is immediately usable.

The multi-agent argument is the most important and least developed section. "Strongly typed interfaces between agents" is the right instinct but the implementation is underspecified. What does a typed agent interface look like in practice? Schema validation on tool outputs? Structured handoff documents with mandatory fields? This needs the same treatment [[The Agentic Product Standard v2.0]] gives to composition patterns — concrete, testable, falsifiable.

The article's deepest insight might be the one it states most casually: "Large invisible deltas are where both humans and agents lose the plot." Small, visible, frequent integration — the XP insight — is the structural defense against spec failure. Everything else is optimization within that constraint.

---
*Sources: [[raw/why-ai-coding-agents-still-need-clear-specs]]*
*Last updated: 2026-07-11*
