# SDD Case Study — 13 Apps in 70 Days

Felipe Fontoura's field report from building a production-grade crypto payment platform solo: 13 apps, 138K lines of TypeScript, 1,650 tests, real money, 70 days. The agent was Claude Code. The differentiator was a 28-file spec corpus that served as external memory across stateless sessions. This is the most concrete public evidence yet for [[Specifications as the Product]] — not a theory piece, but a shipping system where the spec, not the code, was the durable artifact.

---

## Key Quotes

> "Capability amplifies direction. It does not supply it."

Fontoura's distillation of the METR finding that experienced developers with AI were 19% *slower* than without it. The AI is a multiplier; if you multiply zero direction, you get zero results. This is the same insight as [[Optimizing for Decision Points]]: the human's job is setting the vector, not writing the code.

> "A single developer does not hold a 13-app fintech in their head. The spec holds it. The developer holds the spec."

The cleanest articulation of why SDD isn't bureaucracy — it's cognitive offload. The spec corpus is the developer's extended working memory. This echoes [[Engineering for Bounded Cognition]]'s ~4-chunk working memory limit and [[Agent Memory and Context]]'s context-as-RAM metaphor.

> "Patch the code, leave the spec untouched, and the next time that module is regenerated or extended, the agent rebuilds from the spec and reintroduces the same error."

Step 4 of the delivery loop — "correct the spec, not the code" — is where the discipline lives. This is the same structural insight as [[The Oracle Is the Asset]]: the spec is the source of truth the compiler (agent) answers to. Code patches that don't flow back to the spec are technical debt with a half-life measured in regeneration cycles, not months.

> "Without the specs, the agent is a fast bricklayer with no blueprint. With them, it is a senior engineer with perfect recall of every decision you made."

The spec as prosthetic memory. Every agent session starts cold; the spec corpus is the warm handoff. This aligns with Anthropic's own guidance for Claude Code (explore → plan → implement → commit) and [[Claude Code Mastery]]'s thesis that CLAUDE.md and skills are compounding infrastructure.

## Key Themes

#spec-driven #case-study #fintech #agentic-coding #cognitive-offload #external-memory #claude-code

## The Delivery Loop

Fontoura's four-step loop per capability: **Specify** → **Generate** → **Verify** → **Correct the spec**. The fourth step is the innovation — it's a self-tightening feedback loop that makes the spec corpus improve with every bug found. This is [[Guardrails and Feedback Loops]] applied at the specification layer, not the code layer.

The loop is not new in structure — [[SDDW (Spec-Driven Development Workflow)]] has a similar 7-step pipeline — but Fontoura's insistence on fixing the spec rather than the code is the differentiator. Most practitioners patch the code and call it done, degrading the spec corpus over time.

## What the Specs Actually Caught

Two examples stand out:

**Pricing precision.** The spec mandated 8-decimal integer arithmetic (satoshi precision), with float usage anywhere being a test failure. Without the spec, the agent would have defaulted to JavaScript's `number` type and introduced rounding drift that only surfaces at reconciliation time — the worst possible moment.

**Payment idempotency.** A `UNIQUE(merchant_id, idempotency_key)` constraint was written into the spec before any charge endpoint code existed. Without it, idempotency lives only in the developer's head and vanishes on the next regeneration. This is [[Constraint Decay]] in reverse: the spec *prevents* the constraint from decaying across sessions.

The author's explicit out-of-scope lists also stopped the agent from adding unrequested features — twice. Agents are helpful to a fault; negative scope is as important as positive requirements. This connects to [[AI Agents Need Clear Specs]]'s finding that the cost curve is U-shaped: zero spec burns tokens on correction loops; well-structured acceptance criteria sit at the minimum.

## The Limitations Fontoura Actually Admits

This is what makes the piece credible. Fontoura lists six limitations without prompting:

1. **N of 1** — one system, one developer, one deadline. No control group.
2. **25 years of experience** — SDD scaled a mature mental model, not a blank slate. "SDD is a force multiplier, and zero times anything is zero."
3. **No independent quality audit** — "Green tests and tight CI have shipped bad systems before."
4. **External deadline** — real motivation, not a lab condition.
5. **Engineering method, not market proof** — "Production is not a business."
6. **Single case, not a methodology** — favorable outcome ≠ broad empirical support.

This is the right way to present a case study. Compare with [[Dev Machine Foundry]] (Sam Schillace's 39-day Word clone), which shares the same honesty about what a single data point can and cannot prove.

## The Spec as External Memory

Fontoura's structural argument matters more than his case study: "an AI agent has no memory between sessions, and the spec is the external memory you give it." This frames SDD not as process discipline but as a solution to a technical constraint — agent statelessness. The spec corpus is the filesystem the agent's RAM pages to.

This connects directly to [[Maybe Coding Agents Don't Need a Bigger Memory]]'s argument that "bigger context windows don't solve cold starts; repo-local, evidence-weighted continuity records do." Fontoura's 28 spec files are exactly that: evidence-weighted continuity records, loaded as context each session.

## Critical Analysis

**The strongest claim is the structural one, not the empirical one.** Fontoura's argument that specs solve agent statelessness is airtight and generalizes. His claim that SDD enabled 13 apps in 70 days is credible but unreproducible — we can't disentangle the method from the practitioner. The article knows this and says so. That's its strength.

**The missing dimension is spec maintenance cost.** Fontoura wrote 28 specs across 12 domains, plus 8 custom slash commands. He doesn't quantify how much of the 70 days went to spec authoring vs. spec maintenance vs. waiting for agents. The final line — "The code wrote itself. The specs did not. That is where the seventy days went" — gestures at this but doesn't break it down. A time budget would make the case study much stronger.

**The "correct the spec, not the code" discipline is the real moat.** Most teams will fail here. Patching code is faster and more satisfying in the moment. The spec-correction loop requires the same deferred-gratification discipline as writing tests before code. Teams that skip step 4 will accumulate spec rot and wonder why SDD stopped working.

**The comparison table is the article's most persuasive artifact.** "The agent's capability was identical in both columns. The spec is the only variable." This is a controlled thought experiment dressed as a field report, and it works because the before/after columns are concrete and falsifiable.

**Fontoura's experience is both a confound and the point.** "SDD scaled a mature mental model" — yes, and that's the whole thesis. The spec *is* the mental model, externalized. A junior developer can't write a 28-file spec corpus for a fintech platform because they don't have the mental model to externalize. SDD doesn't replace expertise; it scales it. This is the same dynamic as [[What You Bring to AI Determines the Result]]: AI is a medium, not a solution.

---
*Sources: [[raw/spec-driven-development-case-study]]*
*Last updated: 2026-07-18*
