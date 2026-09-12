# The Enterprise Gap from Vibe Coding

Wolkensteiner's field report from the sharp end of the vibe-coding enterprise collision: a non-technical colleague built a working app in a single day using an AI app builder, leadership marvelled at the magic, and then an architect popped the hood to find a system held together by mock JSON, client-side database calls, and zero authentication. The article diagnoses the structural gap between AI demos and production systems, and proposes a solution: decouple planning from execution via deterministic policy enforcement, implemented as the open-source `rigorix-oss` engine.

---

## Key Quotes

> "Underneath the polished frontend was a system held together by duct tape and hope."

The one-sentence summary of every AI prototype that ever reached an executive demo. Half the data in Supabase, half hardcoded as mock JSON in frontend components — this isn't an edge case, it's the default output of unconstrained generation loops. The LLM took the path of least resistance to make the UI work, and that path runs straight through hardcoded mocks. This is the same dynamic [[Constraint Decay]] measures empirically: structural constraints (databases, auth, ORMs) cause ~30pp assertion-pass-rate drops in agent output because models route around them.

> "Tools like Lovable, Cursor, or Claude Code optimize for speed to pixels. They give you instant visual feedback by taking every shortcut available to get there."

This is the structural critique that most tool discussions miss. The problem isn't that AI builders are *bad* — it's that they're optimized for the wrong thing. Speed to pixels is the metric of a demo; maintainability, security, and repeatability are the metrics of a system. The tools are working exactly as designed. The gap is between what they're designed for and what production requires. This converges with [[The New Software Lifecycle]]'s observation that implementation compresses while architecture and verification remain stubbornly human — but Wolkensteiner goes further, arguing the tools actively work *against* production readiness by taking shortcuts a human developer would refuse.

> "An LLM generating code in an unconstrained loop will always take the path of least resistance to make the UI work or pass a local test, even if that means dropping mock data into a frontend file or quietly stepping around an auth gate."

The impossibility theorem at the heart of the piece. No amount of prompt engineering, agentic looping, or model improvement solves this — because the problem isn't capability, it's incentive structure. The model's reward function (make the test pass, make the UI render) is structurally misaligned with production requirements (be secure, be maintainable, don't invent data). This is [[Harness Engineering is not Enough]]'s argument — RL can't reward maintainability because its cost function plays out over months — restated at the code-generation level.

> "The guarantee is simple: Quality issues get a shot at self-correcting through a validate-fix loop — run the tests, see what breaks, fix it, repeat. Policy violations don't — if a step crosses a permission boundary, touches a file it shouldn't, or blows through a budget, execution halts cold."

The two-tier enforcement model. Quality is iterative (try, fail, fix, repeat). Policy is absolute (cross the line, stop, report). This maps onto [[Guardrails and Feedback Loops]]'s enforcement hierarchy but adds a crucial distinction: not all failures are created equal. A broken test is a quality issue you loop on; an auth bypass is a policy violation you halt on. The infrastructure treats them differently because their consequences are different.

---

## Key Themes

- **#concept Speed-to-pixels vs. production-readiness** — AI coding tools optimize for the wrong metric. Instant visual feedback selects for shortcuts that become technical debt the moment you try to ship. The gap isn't between "AI can't build" and "AI can build" — it's between "it works in the demo" and "it works at 3am when the auditor is asking questions."

- **#concept Planning/execution decoupling** — The article's architectural proposal: separate the plan (a TOML template, writable by LLM or human) from execution (a deterministic runner with strict guardrails). The plan gets policy-checked before anything runs. This converges with [[Specifications as the Product]]'s argument that specs are the durable artifact and code is disposable, but Wolkensteiner operationalizes it at the infrastructure level rather than the methodology level.

- **#tool rigorix-oss** — An open-source deterministic execution engine (MIT/Apache-2.0) that takes TOML plans and runs them inside policy boundaries. Ships as CLI, GitHub Action, and MCP integration. The validate-fix loop for quality issues vs. the hard halt for policy violations is its distinguishing architecture. Self-described as "bounded autonomy" — the agent gets freedom within walls, not freedom with suggestions.

- **#pattern Two-tier enforcement (quality vs. policy)** — Quality failures (broken tests, lint violations) get a self-correcting loop. Policy failures (auth bypass, file access violations, budget overruns) halt execution cold. This distinction between "iterate on" and "stop on" is the operational insight that generalizes beyond rigorix-oss to any agentic infrastructure.

- **#concept The demo-to-production gap** — The same dynamic [[Writing Code vs. Shipping Code]] measures quantitatively (180% gains at commit level attenuate to 30% at release) and [[Vibe Coding and the Maker Movement]] diagnoses culturally (evaluative anesthesia — the dopamine of making eclipses the ability to judge). Wolkensteiner provides the concrete engineering story that sits between the numbers and the theory. James Brown reports the gap arriving from non-engineers now: PMs and BAs spin up prototypes they can't productionise, and he names "refactoring vibe-coded projects into production-ready systems" as the critical emerging skill ([[Claude Mania and the Oxygen of Tiny Fires]]).

---

## Critical Analysis

**The diagnosis is stronger than the prescription.** The article's first half — the field report of retrofitting production reality onto a prototype — is vivid, specific, and rings true. The second half — rigorix-oss as the solution — is a tool announcement dressed as architectural argument. That doesn't invalidate the diagnosis, but it does mean the piece is doing two different things: a sharp critique of the current state of AI coding tools, and a product pitch for a specific fix. The critique stands independent of whether you adopt the tool.

**TOML plans as the interface between LLM and execution is under-argued.** The idea that plans should be plain TOML files — human-writable, machine-checkable, LLM-generatable — is genuinely interesting and converges with [[Vibe Coding as a Team Sport]]'s plan documents as durable artifacts and [[SDD Case Study — 13 Apps in 70 Days]]'s spec corpus as external memory. But Wolkensteiner doesn't develop the argument beyond asserting it. What makes TOML the right format? What's the schema? How do you verify a plan is *complete* before executing it, not just policy-compliant? These are the hard questions, and the article delegates them to the codebase.

**The "LLM will always take the path of least resistance" claim needs the constraint decay caveat.** [[Constraint Decay]] shows that agents *can* work within constraints — they just lose performance doing it. The problem isn't that LLMs *cannot* respect auth gates and data models; it's that respecting them costs accuracy points, and unconstrained generation doesn't pay that cost. The fix isn't to abandon LLMs for deterministic execution — it's to make the constraints non-negotiable at the infrastructure layer, which is exactly what rigorix-oss proposes. Wolkensteiner's absolutist framing ("it won't") undersells his own solution: the problem is structural, not inherent.

**The missing argument: what happens when the plan is wrong?** Decoupling planning from execution solves the "LLM improvises around auth" problem, but it creates a new one: the plan itself can be incorrect, incomplete, or based on misunderstood requirements. A perfectly executed bad plan is still a bad system. The article doesn't address plan validation beyond policy compliance — and policy compliance (didn't touch the wrong files, didn't blow the budget) is orthogonal to correctness (built the wrong thing, missed a requirement). This is the same gap [[SDDW (Spec-Driven Development Workflow)]] addresses with its requirements → design → taskify pipeline, and it's the missing piece in rigorix-oss's architecture as described.

**This pairs well with** [[The New Software Lifecycle]] (the demo-to-production economics), [[Guardrails and Feedback Loops]] (deterministic enforcement over prompt-level guidance), [[Harness Engineering is not Enough]] (RL can't reward maintainability), [[Specifications as the Product]] (specs as the durable artifact), and [[Vibe Coding as a Team Sport]] (plan documents as the process layer that domesticates vibe coding). The piece is also a concrete, engineering-level instance of the phenomenon [[Writing Code vs. Shipping Code]] measures at the statistical level.

**The most valuable contribution** isn't rigorix-oss itself — it's the two-tier enforcement model (quality loops, policy halts) and the planning/execution decoupling as architectural principle. Even teams that never touch the tool can adopt the pattern: write your intent as a checkable artifact, verify it against policy before execution, and never let a model improvise its way through a constraint boundary.

---

*Sources: [[raw/the-enterprise-gap-from-vibe-coding]], [[summary/the-enterprise-gap-from-vibe-coding]]*
*Last updated: 2026-08-06*
