# Agent Skills Library (dzhng)

A curated library of 19 domain-agnostic, composable agent skills that together form an opinionated workflow for autonomous software development: plan, slice, build, verify, repeat. The skills treat unknowns as fog of war to be mapped, specs as living documents that rewrite themselves mid-implementation, and every slice as something that must survive three independent reviews before the loop advances. One reported unattended Codex run lasted 1 day 16 hours — the skills as a software factory operating system.

---

## Key Quotes

> "The spec is a living document, updated and re-sliced mid-implementation."

This is the core inversion. Traditional specs freeze before implementation; here the spec *learns* from implementation. When the agent discovers the plan is stale, it rewrites the spec rather than pressing on with a wrong map. Each iteration gets "less wrong, until the goal is done." This is [[Specifications as the Product]] in its purest operational form — the spec isn't documentation, it's the control surface.

> "Before the loop advances, every piece must survive architecture review, code review, and visual review against a stored baseline."

Three independent verification gates per slice. Not "review when you remember" — mandatory, automated, enforced by the spec's own instructions to the agent loop. This is [[Guardrails and Feedback Loops]] applied at the slice level: deterministic enforcement, not prompts pleading for quality.

> "Each skill is a standalone folder others can copy or customize. Domain-agnostic. Harness-agnostic. Works with Claude Code, Codex, opencode, Cursor, duet, and 70+ others."

The library's portability thesis: skills shouldn't be coupled to any one harness. This is a bet that the skill format (markdown prompts in folders) is the right unit of reuse, and that [[Team-Wide Agentic Harness|version-controlling skills as team infrastructure]] is the right social contract. The `npx skills add` distribution mechanism reinforces this — skills as packages, not copy-paste.

## Key Themes

- **#pattern Spec as control surface**: The spec isn't a requirements document — it's an executable control loop that decides when to invoke review skills, re-slice work, and close out. This is [[SDDW (Spec-Driven Development Workflow)]] without the formal seven-step pipeline — same insight, lighter implementation.

- **#pattern Slice-verify-repeat**: The atomic unit of work is a verifiable slice, not a task. Every slice faces three review dimensions (architecture, code, visual) before the loop advances. This is the verification discipline that [[Agent Coding Workflow]] describes as the line between vibe coding and engineering.

- **#pattern Multi-agent as default**: The `codex` and `claude` skills treat second agents as standard infrastructure — not a special case. Claude Code and Codex CLI are used as independent reviewers and delegated implementers, implementing the planner/worker/judge pattern from [[Agent Orchestration]].

- **#tool Skills as packages**: `npx skills add` distribution treats skills as installable dependencies with a registry, versioning, and selective installation. This is the same pattern as [[sx]] (team package manager for AI coding assets) and represents the packaging layer of [[Loop Engineering]].

- **#concept Fog-of-war methodology**: The `explore-unknowns` skill maps task unknowns across four quadrants before any code is written. This is [[Load-Bearing Assumptions]] applied to task scoping — surface what you don't know before you commit to a plan.

- **#pattern Tracer-bullet testing**: `write-tests` produces tests that "pin real behavior, not implementation details." This is the testing philosophy from [[The Oracle Is the Asset]] — the test suite as durable artifact, not disposable verification. It also echoes [[Guardrails and Feedback Loops]]'s insistence on deterministic enforcement over aspirational prompts.

## Critical Analysis

**The library is a rare example of tasteful curation in a space addicted to maximum surface area.** Nineteen skills, one clear workflow, no feature creep. Each skill does one thing and composes with the others. The `review` skill (combining refactor-clean, code-review, and write-docs) shows the author understands that *fewer invocation points beat more granularity* when the agent is already context-saturated. This is the opposite of the everything-tool approach that bloats most agent frameworks.

**The "fog of war" framing is the library's most underrated idea.** Most agent workflows assume a correct plan exists and the challenge is execution. dzhng starts from the opposite premise: you don't know the terrain, the plan will be wrong, and the system must *discover* the right plan through implementation. The `explore-unknowns` skill (four quadrants: known knowns, interviews, reactable artifacts, blindspot passes) is a lightweight but rigorous epistemology for agentic work. Combined with `write-spec`'s mid-implementation re-slicing, this creates a feedback loop that's structurally resistant to the stale-plan problem that kills most autonomous runs.

**The 19-skill count is a feature, not a limitation.** Compare to the 16-skill [[Agent Skills for Security Testing|security testing library]], the 11-skill [[Audit Skills for AI Coding Agents (metacircu1ar)|audit skills library]], the sprawling 7-step [[SDDW (Spec-Driven Development Workflow)]], the 10-component [[Loop Engineering]] taxonomy, or the 8-skill [[PAAD — Defense-in-Depth for AI-Assisted Development]] suite (which shares dzhng's composable-skills philosophy but targets defense-in-depth quality gates rather than autonomous development workflow). dzhng's library sits at exactly the right level of abstraction: enough skills to cover the full lifecycle, few enough that an agent can hold the catalog in context. The `npx skills add --list` flag (pick individual skills) acknowledges that most users will take 3–5, not all 19.

**The library's main gap is evaluation.** The `eval-skills` skill is meta — it evaluates the skills themselves, not the code the skills produce. There's no equivalent of [[Razorback]]'s reproducible benchmarking, no pass@k measurement, no regression suite. For a library that makes verification its central thesis, the absence of quantifiable quality metrics for its own output is a conspicuous silence. This is less a criticism of dzhng and more a reflection of where the field is: we're still at the "eyeball the results" stage of skill evaluation, and `eval-skills` (blind runs, fresh subagents, separate judge) is more rigorous than most.

**The 1d 16h unattended run is both the proof and the warning.** An agent running autonomously for 40 hours on a single goal demonstrates that the spec-loop architecture works at scale. But it also demonstrates the [[Dev Machine Foundry|Dev Machine Foundry]] pattern's structural risk: priority inversion, where optimization-driven agents chase local maxima while missing the strategic goal. The `close-spec` skill's rewrite of the build plan into "a durable rationale record" is the hedge against this — it forces the agent to explain what it built and why, creating a human-auditable artifact from an autonomous run.

---

*Sources: [[raw/dzhng-skills]]*
*Last updated: 2026-07-21*
