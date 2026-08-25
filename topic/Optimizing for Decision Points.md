# Optimizing for Decision Points

Shawn Simister's framework for identifying the critical moments in agent-assisted development where human judgment has outsized impact — and designing workflows that surface those moments rather than letting models fill them with safe, generic defaults.

---

## Key Quotes

> "A few minutes of human judgment at the right time saves hours of agent work heading in the wrong direction."

This is the thesis in one sentence. It's not about doing more work — it's about doing the *right* work at the *right* time. The corollary is that most of what we currently review (code, not decisions) is low-leverage.

> "The agent's job isn't to have taste on your behalf, but to notice the moments where taste matters and put them in front of you."

This flips the standard agent-design assumption. We've been trying to build agents that make better decisions autonomously. Simister argues the real engineering problem is building agents that reliably *recognize* when a decision needs a human — which is a simpler, more tractable problem than making the decision itself.

> "The review feels productive, and it is, but the decisions that had the most influence over what you're reading happened before any code was written."

A brutal reframe of code review in the agent era. Simister channels Donella Meadows here: code review operates at the parameter level (low leverage), while the decisions that shaped the code were made earlier at the system-rules level (high leverage). Reviewing agent-written code without having shaped the upstream decisions is inspecting the rivets on a bridge someone else designed.

> "Anyone using the same models can go from a half-formed idea to a working prototype in minutes. What will differentiate your work is a willingness to push beyond what models produce by default."

The democratization of "good enough" means differentiation comes from taste, not speed. Models converge on the safe median — your job is to push past it.

## Key Themes

### Decision Points as the Scarce Resource

Simister identifies a new bottleneck in agent-assisted development: not coding speed, not model capability, but *human attention at decision points*. As agents get faster, the premium on knowing where to intervene goes up. This is a #pattern for thinking about workflow design.

The taxonomy of decision levels maps cleanly onto the development lifecycle:
- **Idea level** (brainstorming) — what to build
- **Design level** (planning) — how to build it
- **Task level** (decomposition) — how to sequence the work
- **Implementation level** (coding) — the actual code

Each level offers a chance to catch assumptions before they compound. The trick is choosing the right depth for each task — over-specifying trivial work wastes attention; under-specifying complex work bakes in bad defaults.

### The Dagre Example as Pattern

The anecdote about the dagre library is the article's most concrete teaching moment. Agent hits a CJS/ESM import problem → agent defaults to shims and workarounds → human asks "are we even using the right package?" → switch to @dagrejs/dagre → problem vanishes at the source.

This is the #pattern: when an agent proposes a workaround, ask what decision *upstream* made the workaround necessary. Don't fix the symptom; reframe the problem. This is the agentic equivalent of "don't debug the code, debug the assumption."

### Meadows' Leverage Points Applied to Software

Simister's application of Donella Meadows' "Leverage Points: Places to Intervene in a System" to software development is the article's deepest insight. The hierarchy:
1. **Parameters** (lowest leverage) — code style, variable names, test coverage percentages
2. **System rules** (high leverage) — architecture, dependency choices, API contracts
3. **Mental models** (highest leverage) — what problem you're solving, who the user is, what "good" looks like

Most agent workflow design focuses on parameters (did the code compile? do tests pass?). Simister argues for shifting attention upward — toward the decisions that *produce* the code. This connects directly to [[Discovery Debt]] (untested assumptions compounding invisibly) and [[Load-Bearing Assumptions]] (surfacing falsifiable claims before they become architecture).

### Escalation as Agent Capability

The `AskUserQuestion` pattern — where an agent recognizes ambiguity and surfaces a specific question — is framed as a design primitive, not a failure mode. The agent doesn't need good judgment to be useful; it needs good *meta-judgment* (knowing when its judgment is insufficient). This inverts the [[Guardrails and Feedback Loops]] framing: instead of catching agent mistakes post-hoc, design the workflow to prevent them at decision boundaries.

[[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] gives this the missing urgency: the cost of *not* building that meta-judgment isn't merely a safe default baked in — it's a decision made with no author and no record, a debt entry that never surfaces at all. Simister answers *how* to surface decision points; bl00cyb explains *why the silent ones are the expensive ones*.

Simister's insight about emergent patterns (prefer ESM-compatible libraries, prototype before testing, use streaming events) becoming reusable agent rules is essentially [[Loop Engineering]] in microcosm — encode taste into the system so you can reserve your attention for genuinely novel decisions.

### Atelier and the Kanban-as-Interface

Simister built [atelier.dev](https://atelier.dev) as an AI-native kanban board where columns represent decision states rather than work states. Each column transition triggers a custom agent skill. This is the #tool embodiment of the decision-points philosophy and connects to the broader pattern of kanban boards as human-agent interfaces (see [[Agent Orchestration]]). It's also a worked example of [[Automating Myself Out of Development]] — building tooling to manage the attention bottleneck.

## Critical Analysis

**What's genuinely novel:** The Meadows framing is the article's real contribution. Everyone talks about "human in the loop," but Simister gives it topology — some loops are near the parameter surface (low leverage), others near the system-rules core (high leverage). The practical implication is that *where* you insert human judgment matters more than *how much* judgment you apply.

**What's underdeveloped:** The taxonomy of decision points is gestured at but not systematized. Simister mentions that matching tasks against a taxonomy of common decision points "is something agents can do reliably," but doesn't provide that taxonomy. Without it, the framework risks becoming another thing that requires taste to apply — which is exactly the bootstrapping problem it aims to solve.

**What's missing:** The article doesn't address the social dimension. Decision points in a team context aren't just about one person's attention — they're about who gets to decide, when decisions become visible to others, and how decisions get revisited. The Atelier tool hints at this (shared kanban board) but the article frames it as a personal productivity problem.

**The tension with spec-driven development:** Simister is essentially arguing against the pure [[SDDW (Spec-Driven Development Workflow)]] model, where you spec everything upfront and the agent implements. His position is more nuanced: some decisions should be specified, some should emerge during implementation, and the art is knowing which is which. This is closer to [[Manifest-Driven Development]] in spirit — let the agent discover what needs deciding by doing.

**The dagre story is doing a lot of work:** It's the article's best example but it's also the easiest case — package selection is a discrete, technical decision with a clear correct answer. The harder decision points (design philosophy, user experience, business logic) don't resolve as cleanly. The framework needs more examples from the messy middle.

**Bottom line:** This is a #concept that deserves to become infrastructure. The question isn't whether decision points exist — they do — but whether we can build tools that reliably surface them. Simister's contribution is naming the problem clearly and sketching a solution direction. The hard engineering work of building the taxonomy and tooling remains.

---
*Sources: [[summary/decision-points]]*
*Last updated: 2026-07-05*
