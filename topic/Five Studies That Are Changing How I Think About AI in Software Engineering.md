# Five Studies That Are Changing How I Think About AI in Software Engineering

Brian Houck's survey of five 2026 papers that converge on a single uncomfortable story: AI is compressing the upstream work of software engineering, and everything downstream is breaking. The papers span productivity measurement, shipping economics, longitudinal DevEx, developer wishlists, and debt taxonomy — different methods, same conclusion.

Houck is also co-author (with Butler, Storey, Lowdermilk, Clarke, and Murphy-Hill) of [[Eight Myths of AI in Software Engineering]], which provides the upstream diagnosis: the eight misconceptions that lead organizations to deploy AI in ways that cause the downstream breakage surveyed here. Read together, the two pieces form a complete arc — myths → bad adoption → shipping attenuation, experience erosion, and compounding debt.

---

## The Five Studies

### 1. Copilot's dose-response effect (Heilman, Kyllo, Murphy-Hill)

Self-comparison over 43 weeks across 16,223 developers: the highest Copilot usage weeks show ~40% more completed PRs per coding hour. Seven robustness tests, holds up. Critically: strongest effects on **larger** PRs (7+ files), killing the "just slicing work smaller" theory.

### 2. The shipping attenuation (Demirer, Musolff, Yang)

Already covered in [[Writing Code vs. Shipping Code]]. Autonomous agents → +180% commits → +50% projects → +30% releases. The production hierarchy filters AI output at every step. **Elasticity of substitution: 0.25** — AI and humans are still complements, not substitutes. This is the paper that should kill "10x developer" discourse.

### 3. The productivity-experience paradox (Vella, Blincoe)

Six-month longitudinal, 95 engineers. Productivity perceptions stayed high (84% report improvement), but the share reporting **worse DevEx** nearly doubled (14% → 27%). Flow state eroded most. **Cross-sectional correlations between DevEx and productivity were strong — but change scores didn't correlate.** Productivity and developer experience are decoupling over time in AI-assisted workflows. This is the finding most likely to be ignored because it's inconvenient to every tool vendor and "AI transformation" initiative simultaneously.

### 4. What developers actually want (Choudhuri et al.)

860 Microsoft developers across roles, domains, and geographies. 22 AI tools they want beyond code generation. Three themes:
- **The "right-shift" burden:** faster coding → more code to review, more incidents to debug, documentation that rots faster
- **Shift to verification:** tools that assemble incident case files, catch business-logic bugs in PRs, generate change-aware tests
- **Bounded delegation:** developers want AI on assembly work (docs, edge-case tests), **never** on core logic, architecture, or critical decisions. The line was drawn even where developers acknowledged AI *could* do the work — this isn't about capability, it's about control.

Four non-negotiable guardrails: authority scoping, data provenance, uncertainty signaling, least-privilege access.

### 5. Cognitive debt and intent debt (Storey)

Houck calls this the most important paper he's read in a long time. I agree.

> "Treat understanding as a deliverable."

Storey argues the technical debt metaphor is insufficient. AI helps with technical debt (refactoring, tests, review) while quietly accelerating two other forms:

- **Cognitive debt** — lives in people. When AI generates code, developers skip building the mental model they'd have built writing it themselves. Scaled across a team over time: "an accumulation of not knowing."
- **Intent debt** — lives in artifacts. Goals, constraints, and rationale are unclear, unwritten, or forgotten. As AI writes more code, intent debt becomes a **first-order constraint** on what AI can do next.

The three debts compound: intent debt → cognitive debt → technical debt → amplifies cognitive debt. This is the taxonomy that explains why AI-assisted codebases feel different from hand-built ones even when all tests pass.

[[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] sharpens the intent-debt half from the process side: the *moment* intent debt is born is a silently-resolved ambiguity — an issue underdetermines a behavior, the agent picks something statistically probable, and no artifact records that a choice was made at all. Storey's intent debt is the accumulation; bl00cyb names the mechanism by which it accrues invisibly.

---

## The Synthesis

Houck's narrative arc, which I find convincing:

> Heilman shows per-engineer gains are real. Demirer shows they don't survive to shipped software. Choudhuri shows the bottleneck moved downstream and developers know it. Vella shows the lived experience is eroding even while throughput holds. Storey shows the deepest cost is unmeasured — the slow erosion of shared understanding.

The bottleneck has shifted from writing code to understanding it, verifying it, and shipping it. **But our tools, metrics, and team designs haven't moved yet.** That gap is where the next several years of work will happen.

---

## Critical Analysis

**What this gets right.** Houck's strength is synthesis — he's not just summarizing five papers, he's telling a single story across them. The narrative arc (gains are real → but don't survive shipping → the bottleneck moved → experience is eroding → understanding is the unmeasured cost) is more valuable than any individual paper. This is what good science communication looks like: the papers become characters in a plot, not items in a bibliography.

**The disclosure is honest in a way most synthesis isn't.** Houck co-authored Study 4 and knows the authors of three papers. He says so upfront. This doesn't invalidate his takes — if anything, insider knowledge makes the synthesis sharper — but it's worth noting that the convergence he sees may partly reflect a shared research community, not just independent confirmation.

**The "productivity-experience paradox" is the most destabilizing finding and the one most likely to be memory-holed.** Every vendor narrative assumes productivity ↑ → DevEx ↑. Vella and Blincoe show the opposite is happening. If this replicates at scale, the entire "developer productivity" measurement industry (including Houck's employer, DX) has a measurement validity problem. Flow state eroded while feedback loops improved — which suggests the problem isn't "AI makes everything worse" but something more specific: AI disrupts the deep-work experience that developers value most, even as it removes the friction they value least.

**Storey's debt taxonomy is the most important conceptual contribution here.** "Cognitive debt" and "intent debt" name things everyone building with AI has felt but couldn't articulate. The key insight is that these debts compound — you can't fix cognitive debt by refactoring code, and you can't fix intent debt by documenting what the code does (you need what it was *supposed* to do). This is the intellectual foundation for why [[Specifications as the Product]] matters: the spec is the intent artifact that resists decay.

**What's missing.** None of these papers address the compensation question. If AI makes coding faster but the human work shifts to review, verification, and understanding — work that is more cognitively demanding and less intrinsically rewarding — do salaries rise or fall? [[Human-in-the-Loop is Tired]] names the exhaustion; these papers quantify the shift that causes it; nobody has yet connected the dots to labor economics.

**The bounded delegation finding should terrify tool vendors.** Developers drew a hard line at "no AI on architecture, core logic, or critical decisions" — not because AI can't do it, but because they don't want it to. This is a preference, not a capability gap. If the market ignores this preference (which it will, because the economics of delegation are too compelling), we get tools nobody asked for doing work nobody wants them doing. The guardrails (authority scoping, provenance, uncertainty signaling, least-privilege) are the right principles, but principles don't survive contact with enterprise procurement.

---

## Key Themes

- **#concept Productivity-experience paradox** — Productivity and DevEx are decoupling in AI-assisted workflows. Flow state erodes while throughput holds. This breaks the assumption that making developers faster makes them happier.
- **#concept Bounded delegation** — Developers have a clear line: AI on assembly work, never on craft. The boundary is about control, not capability. Ignore it at your retention's peril.
- **#concept Cognitive debt** — The "accumulation of not knowing." AI-generated code accepted without understanding, compounding across teams over time. This is why AI-assisted codebases feel hollow.
- **#concept Intent debt** — When goals, constraints, and rationale are unclear or forgotten, AI hits a wall. The spec becomes the first-order constraint on what AI can do.
- **#concept The right-shift burden** — AI moves the bottleneck from writing to reviewing, verifying, debugging, and understanding. Faster generation → more downstream pressure.
- **#pattern Understanding as a deliverable** — Storey's operational principle: shared understanding is a first-class product of development, not a side effect.

---

## Commentary Comment Highlight

> "AI moves the bottleneck from typing code to reviewing it." — Engincan Veske

And asks whether the senior-junior reviewer gap widens. Houck liked it. The question is structural: if AI generates code at junior-to-mid level and senior engineers are the only ones qualified to review it, the bottleneck doesn't just move — it **narrows** into a single point of failure.

---

*Sources: [[raw/five-studies-changing-how-we-think-about-ai-in-se]]*
*Last updated: 2026-07-18*
