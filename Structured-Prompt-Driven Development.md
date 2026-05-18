# Structured-Prompt-Driven Development

SPDD is Wei Zhang and Jessie Jie Xia's methodology for treating prompts as first-class delivery artifacts — version-controlled, reviewed, and kept in sync with code. Built around a 7-part REASONS Canvas and a 6-step workflow, it's the most rigorous "spec-anchored" approach to AI-assisted development published to date. The core insight: individual AI speed doesn't translate to team throughput without structured intent capture. "When reality diverges, fix the prompt first — then update the code."

---

## The REASONS Canvas

The centerpiece of SPDD is a 7-part structured prompt template:

- **R — Requirements:** Problem statement + Definition of Done
- **E — Entities:** Domain concepts and their relationships
- **A — Approach:** Strategy to meet the requirements
- **S — Structure:** Where the change fits in the existing system
- **O — Operations:** Concrete, testable implementation steps
- **N — Norms:** Cross-cutting engineering standards (linting, testing, patterns)
- **S — Safeguards:** Non-negotiable boundaries (don't touch X, don't change Y)

The Canvas functions as both a generation artifact and a review artifact — the reviewer reads the Canvas, not just the code diff, to understand *intent*.

## Key Quotes

> "The real question isn't 'How do we generate more code?' It's how do we make AI-generated changes governable, reviewable, and reusable."

This is the thesis. SPDD is a governance framework, not a productivity hack. The comparison that opens the piece — "buying a Ferrari and driving it on muddy roads" — makes the same point: individual tool speed is worthless without team infrastructure.

> "When reality diverges, fix the prompt first — then update the code."

The golden rule. Two types of divergence: logic corrections flow prompt→code (`/spdd-prompt-update`), refactoring flows code→prompt (`/spdd-sync`). The prompt is always the source of truth for intent, even when code leads on implementation details.

> "The loop is closed by the workflow and the artifact, not by an autonomous learning mechanism."

A deliberate contrast with systems that aim for self-improving agents. SPDD's loop is human-mediated and artifact-driven — the REASONS Canvas is the persistent artifact, and the human closes the loop through review.

> "In the AI era, software development isn't a contest of model IQ. It's a contest of engineer cognitive bandwidth."

The closing line. Models will keep improving; the bottleneck is how much intent a human can articulate and verify. This echoes [[From AI Studio to AI Forge]]'s "the unit is the operating loop" and [[Guardrails and Feedback Loops]]'s feedforward axis.

## Key Themes

#spec-driven #prompt-engineering #workflow #tool #governance #concept

## Critical Analysis

**What SPDD gets right.** The REASONS Canvas is genuinely good design. It separates concerns that most prompt engineering advice conflates: requirements (R), domain model (E), strategy (A), system context (S), implementation (O), standards (N), and constraints (S). This is not a prompt template — it's a specification format. The Operations section (concrete, testable steps) is where the Canvas earns its keep: it forces the human to decompose before the AI generates, which is the single highest-leverage intervention in AI-assisted development.

**The "spec-anchored" positioning is honest.** Zhang and Xia credit Birgitta Böckeler's categorization and explicitly distinguish SPDD from spec-first (write spec, then code) and spec-only (spec is the deliverable). Spec-anchored means the spec *remains* after generation and gets updated bidirectionally. This is [[Specifications as the Product]] operationalized as a workflow rather than a philosophy.

**The 6-step workflow is ambitious for teams that can't even maintain a CLAUDE.md.** Steps 1-6 assume a team that writes requirements, reviews analysis, generates context, generates a structured prompt, generates code, reviews, adjusts, generates tests. Every step has a human checkpoint. This is great for high-compliance environments (the article rates it ★★★★★ for those). For a team doing exploratory work or firefighting, it's ★★. The honesty about fitness is refreshing — most methodology pieces claim universal applicability.

**Where SPDD is underspecified.** The article doesn't address what happens when the REASONS Canvas itself drifts from reality. The two sync commands cover prompt↔code, but neither covers requirements→Canvas. If the product manager changes requirements mid-sprint (and they will), the Canvas-creation workflow re-runs from step 1 with no clear path for partial updates to an existing Canvas. This is where [[OpenSpec]]'s spec deltas have an edge: partial change is the common case.

**The comparison with [[The Lifecycle of a Swamp Issue]] is instructive.** Swamp's five-phase state machine (triage, planning, adversarial review, iteration, implementation) is a process architecture; SPDD's REASONS Canvas is an artifact architecture. Swamp forces the *agent* to follow steps; SPDD forces the *prompt* to have structure. Both approaches converge on the same insight — that AI-assisted development needs gates — but from opposite ends: process vs. artifact.

**What SPDD is not.** It's not compound engineering in the [[Compound Engineering]] sense — there's no automated verification loop beyond human review. It's not a dark factory workflow — every step has a human in the loop. It's not vibe coding's opposite; it's vibe coding's *operating system*. If vibe coding is anarchy, SPDD is constitutional monarchy with a detailed bill of rights.

**The unspoken assumption.** SPDD assumes the hardest part is *capturing intent*, not *verifying output*. For the compliance environments where it rates ★★★★★, this is true — if you can perfectly specify what you want, the generation and review are tractable. For environments where verification is the hard problem (complex systems with emergent behavior), SPDD's Canvas provides intent capture but no verification mechanism beyond human review. [[Guardrails and Feedback Loops]] and [[Harness Engineering]] cover what SPDD leaves to the reviewer.

---

*Sources: [[raw/structured-prompt-driven-development]]*
*Last updated: 2026-05-18*
