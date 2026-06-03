# Thought Refiner Skill

A 15-line Claude Code skill that takes vague user input and turns it into sharp, well-formed questions. Part of a three-skill suite with thought_sharpener (critique) and thought_expander (framings/analogies). The skill is notable not for complexity but for disciplined minimalism: it's clearer about what it *won't* do than most 200-line skill files.

---

## Key Quotes

> "Surface embedded assumptions. Name them plainly."

The first move, not the third. Before asking questions or resolving ambiguities, the skill surfaces what the user is already claiming without knowing it. This is the same instinct behind [[Load-Bearing Assumptions]] — make the invisible visible before acting on it.

> "Flag structural ambiguities. 'Transforming' might mean culture, ops, or comp — which?"

The example is doing real work here. The skill doesn't just say "look for ambiguity" — it demonstrates the move: pick a single word, enumerate the concrete interpretations, ask which one. This is pattern transfer through example, not instruction.

> "A short seed might yield three questions, a long reflection might yield ten or more. No padding."

The adaptive question count is under-specified in a way that's probably correct. It trusts the model to calibrate output density to input density rather than prescribing a fixed number. Most skills over-specify quantity because they don't trust the model; this skill trusts the model.

> "Don't pull in background knowledge about the user unless it's actually load-bearing for the current input."

A sharp insight buried in the engagement style. Most agent skills eagerly load everything they know about the user into context, creating a false sense of personalization that drowns the signal. This skill explicitly gates contextual knowledge on relevance.

## Key Themes

#skill #prompting #clarifying-questions #assumptions #ambiguity #concept

## Architecture of the Skill

The skill has three layers:

**The four moves** (ordered, sequential): surface assumptions → flag ambiguities → propose questions → ask clarifying questions back. Each move depends on the previous — you can't ask good questions before you know what's assumed or ambiguous.

**The five exclusions** (parallel, invariant): no critique, no expansion, no research, no unsolicited suggestions, no flattery. These aren't just scope boundaries — they're delegation points. Critique goes to thought_sharpener. Framings go to thought_expander. Evidence goes to researcher. This is modular skill design: each skill does exactly one thing and explicitly points to siblings for everything else.

**The engagement rules** (meta-layer): no preamble, numbered lists, skip irrelevant context. These make the output scannable — the user shouldn't have to hunt for the questions.

## Critical Analysis

**What's good:** The skill is 25 lines total (15 substantive). It achieves more clarity of purpose in those lines than most skills achieve in 200. The exclusion list is the real design document — it's easier to define what a thing *isn't* than what it *is*, and Davidson leans hard into that. The delegation pattern (thought_refiner → thought_sharpener, thought_expander, researcher) is the right instinct for skill composition: small, single-purpose tools that compose rather than monoliths that do everything.

**What's missing:** The skill says "propose sharp questions" but doesn't define what makes a question sharp. The example ("Transforming might mean culture, ops, or comp — which?") is doing a lot of unstated work. A practitioner who hasn't internalized that example might produce bland questions. This is the trade-off of minimalism: the skill assumes competence from the model.

**The delegation gamble:** The skill delegates critique, expansion, and research to other skills — but those skills might not be loaded. If thought_sharpener isn't available, the skill silently drops that capability rather than falling back to a generic mode. This is either disciplined (stay in your lane) or fragile (the pipeline breaks on missing dependencies). I suspect it's disciplined — a skill that tries to be its own fallback is a skill that loses its crisp edges.

**The "load-bearing context" line deserves its own wiki page.** The instruction to ignore user background knowledge unless it's "actually load-bearing" solves a real problem that almost every agent skill gets wrong: over-contextualization. Agents load everything they know about a user into every interaction, producing creepy-familiar output that's actually *less* useful because it's drowning in irrelevant personalization. This one line is the sharpest thing in the skill.

## Related Pages

- [[Load-Bearing Assumptions]] — surfaces hidden assumptions in code plans; same instinct, different domain
- [[The Car Wash Question]] — clarifying questions suppressed by product design, not model capability
- [[Not-Knowing (Vaughn Tan)]] — diagnosis-before-action framework for uncertainty
- [[Components of a Coding Agent]] — where skills like this fit into agent architecture
- [[Agent Coding Workflow]] — the broader practice this skill serves
- [[Orchestrator - Worker Skill]] — another Claude Code skill; compare structural minimalism

---
*Source: [[raw/thought-refiner-skill]]*
*Last updated: 2026-06-04*
