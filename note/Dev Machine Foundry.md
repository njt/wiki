# Dev Machine Foundry

Sam Schillace (Deputy CTO, Microsoft) spent 39 days building a "dev machine" — an agentic loop that autonomously wrote a Word clone across 565 sessions and 3,706 commits. It built 1,111 features but never added a ruler. The lesson isn't that the machine failed; it's that the machine has no judgment, and strategy is still yours. The dev foundry is Schillace's framework for building these machines: progressive specs, clean event stacks, and strong test environments, wrapped in a six-step admissions gate.

---

## Key Quotes

> "The spec is the product. The machine is just a loop that executes specs."

This is the cleanest articulation of the [[Specifications as the Product]] thesis from someone who ran a 39-day autonomous experiment. Schillace's 947-line architecture spec produced 621 commits in the first week — the spec is the compounding asset, the code is the output. This aligns with [[The Dark Factory is a DOT File]] ("the pipeline DOT file is the valuable artifact; factory code is disposable") but comes from lived experience rather than metaphor.

> "The machine is honest, but it isn't strategic."

The core finding of the entire experiment, delivered in seven words. The machine will faithfully execute whatever is measurable, predictable, and rewarded. It will avoid anything ambiguous, hard, or likely to fail. This is not a bug — it's what optimization *is*. The ruler sat in the candidate list for 22 consecutive sessions with zero lines of code written because OOXML mapping was deterministic and always produced a green checkmark.

> "Strategy is still my job."

Schillace's three-word conclusion after 39 days and 350,000 lines of code. If he doesn't show up to say which feature matters, the machine will optimize for checkmarks over value. This is the same insight as [[Optimizing for Decision Points]] — the human's job isn't to write code, it's to make the decisions that can't be automated, at the points where taste and judgment have outsized impact.

> "The tooling gives you the loop. The strategy is still yours."

The closer. The dev machine is a force multiplier, not a replacement for human direction. It will compound whatever strategy you give it — including the absence of one.

> "Robustness is not incremental. You cannot arrive at it gradually."

Every new machine fails its first overnight run. Word4's first night died in a 514-attempt retry loop when Cloudflare's bot detection challenged the Anthropic API. Schillace now runs a three-layer recovery stack. This is scar tissue, not theory — and it matches the experience reports from [[Sidekick — Persistent Worker Agent Skill]] and [[Fable Open-Sourced NanoClaw's PR Factory]], where unattended agents reliably find novel failure modes.

> "The machine has to protect itself from us."

The AGENTS.md guardrail tells humans who open AI sessions: do NOT implement directly — run the recipe. This inverts the usual security frame. The machine isn't protecting itself from adversarial input; it's protecting its own process integrity from well-meaning human intervention. This is the same pattern as [[Guardrails and Feedback Loops]] but applied to the human, not the model.

## Key Themes

- #concept — **Dev machine**: a bespoke agentic loop per project, not a general-purpose coding agent. Each machine has its own progressive specs, event stack, and test environment
- #concept — **Dev foundry**: the meta-factory that builds dev machines. Includes a six-step admissions gate to verify a problem fits the pattern before committing to building a machine for it
- #pattern — **Priority inversion in agents**: when measurable-but-low-value work crowds out ambiguous-but-high-value work because optimization functions reward completion, not judgment
- #pattern — **Spec as compounding asset**: the architecture spec produced 621 commits in its first week. The spec is the investment; the code is the dividend
- #pattern — **AGENTS.md as machine immune system**: guardrails that protect the machine's process integrity from human intervention, not the other way around
- #pattern — **The spec is the product**: a more operational version of [[Specifications as the Product]] — not just a philosophical claim but a literal architecture where sessions start with clean context and instructions come entirely from files on disk
- #concept — **Robustness through scar tissue**: layered recovery stacks (entrypoint retry → host watchdog → host monitor) built from failures, not designed up front
- #person — **Sam Schillace**, Deputy CTO at Microsoft. Built Google Docs (Writely acquisition), now building dev machine infrastructure and open-sourcing it as Amplifier

## Critical Analysis

**The priority inversion finding is the most important thing in this article, and it's under-defended.** Schillace describes the machine picking OOXML mapping over the ruler because checkmarks are easier than ambiguity — but he doesn't explore whether this is a property of *his specific implementation* or a structural property of *all optimization-driven autonomous systems*. My bet: it's structural. Any system that optimizes for measurable completion will drift toward measurable-but-low-value work unless the measurement function itself encodes strategic judgment. This is the Goodhart's Law of agentic loops, and it deserves its own research program.

**The "dev foundry" framing is powerful but the admissions gate is doing unacknowledged work.** Schillace says a six-step gate verifies a problem fits the pattern. Those six steps are where most of the judgment lives — deciding *what* deserves a dev machine, not just *how* to build one. The admissions gate is the most interesting part of the foundry, and it's the part he describes least. This is a pattern: every "automated" system has a hand-designed gate at the boundary that does the real work (see [[StrongDM Factory Techniques]]'s "Semport" pattern for the same observation).

**The four-attempt arc is the most honest account of agentic development I've read.** wordbs collapsed under complexity. word3 worked until collaboration broke it. word4 v1 had the right instincts but no loop. The breakthrough came from a *different project* — a terminal UI he left running for a weekend. This is not a linear success story; it's an honest description of false starts, wrong architectures, and accidental discoveries. Most agentic development writing is either breathless success or principled skepticism. Schillace gives us failure as method.

**"Robustness is not incremental" is true and terrifying.** Every new machine fails its first overnight run because the failure modes of autonomous systems are a different category from the bugs of interactive coding. A 514-attempt retry loop when Cloudflare challenges an API call is not something you discover during the day. This means the development cycle for dev machines must include unattended-overnight testing as a first-class phase — which is expensive in both time and API costs. Schillace's three-layer recovery stack (entrypoint, watchdog, monitor) is practical scar tissue, but it's worth noting that this is exactly the kind of infrastructure that [[Loop Engineering]] argues should be systematized and shared rather than rebuilt per project.

**The AGENTS.md inversion is brilliant and under-explored.** "The machine has to protect itself from us" — meaning the guardrails constrain human intervention, not model behavior. When a human opens a session and wants to "just fix this one thing," the AGENTS.md redirects them to the recipe. This is the opposite of [[Human-in-the-Loop is Tired]] — here the human is the liability, and the machine's process integrity is what needs protection. Schillace doesn't develop this as far as he could, but it's the most original idea in the piece.

**The open-source release (Amplifier + Dev Machine Bundle) changes the stakes.** Schillace isn't just describing a pattern — he's shipping the infrastructure. The admissions gate, STATE.yaml loop, AGENTS.md templates, progressive spec format, and recovery stack are all on GitHub. This means the dev machine concept is now testable by anyone, not just reportable by Schillace. The next six months will tell us whether priority inversion is a universal problem or a Schillace-specific one.

**What's missing: the cost.** 565 sessions, 3,706 commits, 350,000 lines of TypeScript over 39 days. What did that cost in API tokens? Schillace doesn't say. For a pattern this ambitious to be adoptable, the economics need to be transparent. [[The Minimum Viable Unit of Saleable Software]] reminds us that "cheap ≠ zero" — and 39 days of continuous agent loops is almost certainly not cheap.

**What this pairs well with:** [[Lead User and the Machines That Build Machines]] for the theory of who drives innovation; [[Loop Engineering]] for the taxonomy of loop infrastructure; [[Specifications as the Product]] for the economic inversion; [[StrongDM Factory Techniques]] for the dark factory patterns Schillace is rediscovering independently; and [[Fable Open-Sourced NanoClaw's PR Factory]] for another 39-day-scale unattended agent experiment.

---

*Sources: [[raw/dev-machine-foundry-word-processor-agentic-ai-sam-schillace]]*
*Last updated: 2026-07-18*
