# Scaling LLMs to Larger Codebases

Kieran Gill's framework for scaling LLM-assisted development divides engineering investment into two buckets: **guidance** (the context and environment you give LLMs to increase one-shot success) and **oversight** (the skills, workflows, and automated checks that catch what guidance misses). Published July 2025 as the third installment in a series on LLMs in software engineering, the piece draws on the author's experience at Blueberry Pediatrics and is refreshingly honest about where the verification bottleneck remains unsolved.

---

## Key Quotes

> "One-shotting" is when "an LLM can generate a working high-quality implementation in a single try."

Gill's North Star. Not "LLMs are amazing" but "the most efficient mode is when they nail it on the first attempt." The entire guidance half of the framework exists to maximize this rate. This aligns with Stripe's [[Minions — Stripe's One-Shot Coding Agents]] framing, where the goal is unattended PRs that pass CI in ≤2 rounds. The opposite — rework — "often takes longer than just doing the work yourself," which is the quiet tragedy of most LLM-assisted development: you spent 20 minutes steering an agent when 15 minutes of typing would have done it.

> "Read every line of generated code. Just because you told an LLM to sanitize inputs, doesn't mean it actually did."

The oversight thesis in one sentence. Prompt-level instruction is probabilistic — the same insight that [[claude-ctrl]] crystallizes as "an instruction that lives only in model context is not a constraint." Gill is honest that this means engineering work doesn't disappear; it migrates from writing to verifying.

> "The utility of a model is bottlenecked by its inputs."

Garbage in, garbage out, stated with the precision it deserves. This is the case for prompt libraries and codebase health as engineering investments, not documentation chores. It echoes [[Harness Engineering]]'s feedforward axis: the quality of what goes in determines the ceiling on what comes out.

> "A 3-ton truck with a middle-schooler behind the wheel puts people in the hospital."

On why you can't just automate engineers away. The LLM is the truck — powerful but without judgment. The engineer is the driver. This is the strongest single-image argument in the piece for why oversight investment can't be skipped.

> "Design produces architecture. Architecture is a bet on the future."

Tucked into the oversight section but worth pulling forward. Gill's argument that architectural skill grows through experience (reading code from leaders, replicating masterworks like Thorsten Ball's interpreter) is the human half of the equation that [[The Next Two Years of Software Engineering]]'s employment data gestures at: senior roles hold steady because architecture is a bet, not a generation task.

> "Safety refers to the language's ability to guarantee the integrity of these abstractions"

Quoting Pierce's *Types and Programming Languages*, Gill applies this to the idea of automated oversight: AST-walking scripts that enforce conventions like `_api.py` entry points. This is [[Pre-Commit Lint Checks]] applied to architectural boundaries — not "please follow this pattern" but "you cannot violate this pattern."

---

## Key Themes

#concept #tool #pattern

- **One-shotting** — The metric that matters. Not tokens generated, not time saved, but "did the agent produce working, high-quality code in one attempt?"
- **Prompt library** — The institutional response to the question "what could've been clarified?" A living document, not a one-time setup. Mirrors [[Writing a Good CLAUDE.md]]'s advice and [[CLAUDE.md (Universal)]]'s instruction-budget discipline.
- **Codebase health as leverage** — Clean code isn't aesthetics; it's context compression. A messy codebase starves the LLM of usable signal. The Cursor team's observation that clean code principles apply equally to human and model readers is the engineering argument for refactoring in the agent era.
- **Automated oversight as bumper rails** — Moving design feedback from human review to deterministic checks. The `_api.py` enforcement via AST scripts is a concrete example of what [[Guardrails and Feedback Loops]] calls "mechanical enforcement" — you don't ask the agent to respect the boundary, you make crossing it impossible.
- **Verification bottleneck** — The unsolved problem. Gill is honest that as agent output volume grows, human review capacity doesn't scale. The incomplete ideas (lower QA barriers, encode PR feedback for LLM-assisted review, bake security into defaults) are acknowledgements that this is where the field needs the most work.

---

## Critical Analysis

**What works:** Gill's guidance/oversight split is the cleanest two-bucket framework I've seen for thinking about LLM-assisted development investment. It maps directly onto [[Harness Engineering]]'s feedforward/feedback axes but is simpler and more actionable for a team lead deciding where to spend time. The prompt library as a *living document updated after every near-miss* is a specific, stealable practice that costs nothing to start. The codebase health argument — that clean code is context compression for LLMs — reframes refactoring from a craft virtue to a productivity multiplier, which is the kind of argument that actually gets engineering managers to fund it.

**What's glossed over:** The verification bottleneck section is admirably honest but reads like a list of things that don't work yet. "Encode frequent PR feedback into documentation so LLMs can assist with review" is an idea, not a solution. This is the gap where [[Compound Engineering]]'s 50/50 rule would prescribe heavy investment, but Gill stops short of prescribing — he's diagnosing, not prescribing. Fair enough for a blog post, but anyone implementing this framework will find that oversight is the expensive half and guidance is the cheap half.

**The Django bias:** The `_api.py` pattern is clever for Django monoliths. Teams on microservices, serverless, or non-web stacks will need to find their own entry-point conventions. The principle (action-oriented facades that shield readers from internal complexity) generalizes; the implementation doesn't. Worth reading Alex Krupp's linked post if this pattern resonates.

**What's missing:** No discussion of how guidance degrades under long-context sessions. The prompt library approach assumes fresh context windows. In practice, a CLAUDE.md preloaded with 5 `@prompts/` files is great for session start and increasingly irrelevant by turn 40. [[Agent Memory and Context]] covers the strategies Gill doesn't — context routing, tiered memory, compaction. The two pieces should be read together.

**The series context matters:** Part 1 (on hype mechanisms) and Part 2 (on AI strengths and limitations) provide the intellectual foundation. Without them, the guidance/oversight framework can read as "obvious good practices." With them, it's the synthesis: given what LLMs actually are (choice generators, per Part 2) and how hype distorts adoption (Part 1), here's where to actually invest.

---

## Related Pages

- [[Minions — Stripe's One-Shot Coding Agents]] — One-shotting at Stripe scale: 1,000+ unattended PRs/week
- [[Harness Engineering]] — Feedforward/feedback axes; the engineering theory behind guidance/oversight
- [[Guardrails and Feedback Loops]] — Linters beat prompts; the enforcement hierarchy from soft to hard
- [[Writing a Good CLAUDE.md]] — Prompt library as CLAUDE.md; short, universal, hand-crafted
- [[CLAUDE.md (Universal)]] — Six token-efficient rules for agent context
- [[claude-ctrl]] — "An instruction in context is not a constraint"
- [[Pre-Commit Lint Checks]] — Automated enforcement as production infrastructure
- [[Compound Engineering]] — 50/50 rule for system improvement vs. feature work
- [[Feedback Loop is All You Need]] — The self-tightening loop Gill gestures at
- [[Agent Memory and Context]] — Context management for long sessions; the problem Gill's prompt library doesn't solve
- [[Specifications as the Product]] — Specs as durable artifacts; the guidance half made concrete
- [[The Next Two Years of Software Engineering]] — Employment data: junior roles drop, senior holds steady because architecture is a bet
- [[Smart Models Dumb Pipes]] — End-to-end principle: LLMs own decisions, not execution

---
*Sources: [[summary/oversight-and-guidance]]*
*Last updated: 2026-05-14*
