# The Harness Is the Company

Shrivu Shankar's business-strategy essay arguing that SaaS companies are becoming harnesses — the full wrapper of infra, context, state, and review interfaces around a stateless model — and that building that harness, not the software it produces, is the new core competency.

---

## What it argues

The essay widens "harness" from tool-specific wrappers (LangGraph, Claude Code) to *everything* surrounding a model: integrations, permissions, context pipelines, review loops. Sub-harnesses compose into a "software factory" under a meta-harness that decides what runs when.

The trajectory has four rungs: no harness → individuals *operate* harnesses → individuals *orchestrate* harnesses → harnesses *orchestrate* individuals. At the last rung the company *is* the harness: the product is model output, and the org chart inverts — people are placed wherever the harness extracts the most taste and judgment from them. Humans become components of the harness.

The slop-factory objection fails, he argues, because a good harness is not lights-out: it *spends* human attention deliberately — routing product decisions to customer-facing synthesis, architectural calls to engineering taste-holders, design variants to design taste-holders. He concedes the outer loop is hard to trust agents with today, but bets it becomes trainable once planning-and-review tasks have verifiable rewards.

Strategically: the harness now shapes trust (what must be true to ship), distribution, efficacy, and domain context — the classic differentiators. In-house AI dev tools at Ramp/Stripe/DoorDash are the leading edge. Companies should own the top-level harness and plug vendors into sub-workflows; if a third party can run your whole outer loop, you have been commoditized.

## Key quotes

> "Following this trajectory, you've actually turned your company into a harness. The work to produce the software service has moved from people to the harness."

The central claim, and it's an inversion of the usual framing. Most "AI transformation" writing treats agents as tools inside a company; Shankar treats the company as scaffolding around agents.

> "The relationship between software and org structure has inverted as the org chart becomes a question of where to put people so the harness gets the most taste and judgment out of them. Humans are part of the harness."

The most provocative line. It resolves into his "taste-holder" mitigation — but note what it gives up: if humans exist to season agent output, the human career ladder, craft identity, and accountability structures in [[Own the Outer Loop]] are all being renegotiated, not just augmented.

> "Harnesses go from internal tooling you'd happily buy to something you'd no more outsource than your product-eng org or your GTM team."

This is the strategic moat argument, and it cuts against the entire vendor ecosystem: the [[Harness Engineering for Self-Improvement]] survey treats harness design as a research frontier, but Shankar is saying it's a *competitive* frontier — which implies fragmentation, not consolidation.

## Key themes

#concept #pattern #project

## Analysis

This is a strong essay whose weakness is that it argues from trajectory, not evidence. The four-rung ladder is elegant, but the top rung ("harnesses orchestrate individuals") is asserted as the telos of the curve rather than observed anywhere — even his own examples (Ramp, Stripe, DoorDash in-house tooling) sit firmly on rung two or three. The "we've spent a lot of time trying" concession about the outer loop is doing heavy lifting: the whole strategy rests on a bet that verifiable rewards will tame the outer loop, which is precisely the part of the stack least amenable to verification.

That said, the framing is genuinely useful. It dissolves the build-vs-buy question correctly: own the orchestrator, rent the sub-workflows — the same conclusion Databricks reaches from the cost side in [[Managing AI Coding Costs at Scale]] with its meta-harnesses. And the "humans are part of the harness" line, however cold, is more honest than "AI copilots" language: it admits that org design becomes harness design, which is what essays like [[Cyborgs Will Kill the Corporation]] argue from the labor side.

The headless-software prediction is quietly the most testable claim: if every company owns an outer harness, every vendor tool on core paths must expose agent-facing interfaces — which ties the business argument directly to the MCP-and-tool-protocols story.

## Relations

- Strengthens [[Own the Outer Loop]]: both locate surviving human value in judgment over agent output, but Shankar pushes further into org design — taste-holders as a *role*, not just a loop.
- Complicates [[Harness Engineering for Self-Improvement]]: reframes harness engineering from a research problem into a competitive necessity, with ownership (not capability) as the question.
- Converges with [[Managing AI Coding Costs at Scale]]: Databricks' meta-harnesses and routing are the operational embodiment of Shankar's "own the top-level harness" prescription.

---
*Sources: [[raw/the-harness-is-the-company]], [[summary/the-harness-is-the-company]]*
*Last updated: 2026-10-03*
