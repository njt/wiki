# Who Owns the Roadmap? Six Ways to Position an FDE Business

Maarten van den Veld's taxonomy of Forward Deployed Engineer business models, ranked by who controls the roadmap and who captures the value — from commoditised time-and-materials to the "engineer at the gate" who holds decision rights, the control layer, the context layer, and accountability.

---

Van den Veld starts from an observation most FDE evangelism skips: AI makes deploying engineers close to customers dramatically cheaper, but it makes the *same* move cheap for everyone, including your competitors. So the interesting question isn't whether FDEs are viable — it's **who owns the roadmap, and who captures the value created by closing gaps in someone else's product.** His answer is a six-position ladder of defensibility.

---

## The six positions

1. **Time and materials.** You bridge gaps in a vendor's product you don't control. Historically you won by out-hiring; AI dissolves that advantage because everyone has the same models. You're competing around a commodity.
2. **Value-added reseller.** You own middleware and vertical IP around the vendor's roadmap. But a connector that took six months now takes days, and agentic loops swallow vertical workflows. His prescription: own the layer that makes AI "usable, reliable, auditable, predictable, and ultimately — *boring*."
3. **Open-source contributor.** Contributing code isn't capturing value; worse, frontier labs "clio-kleptically ingest" open-source codebases as training data, synthesising your domain edge into model weights. He expects "peek licenses" and dual licensing to emerge.
4. **Strategic partner.** You influence the vendor's roadmap — but influence is not control. The vendor absorbs what you proved and ships it free with the license.
5. **Palantir.** Embed so deeply that removal is existential. For everyone else, the front-house/back-house split (FDEs on commercial realism vs core R&D on architectural beauty) creates product drift and a synchronisation tax only government-scale clients can fund.
6. **The engineer at the gate.** You sit on the buyer's side of the table, holding decision rights over what gets bought, built, integrated, or retired.

## The four moats of Position 6

- **Decision rights** — your incentive is the client's total cost of technology, not a vendor's ARR. Sometimes the answer is buy, sometimes build, sometimes the client should change how they work.
- **The control layer** — model routing, deterministic guardrails, eval harnesses, observability, audit trails. Client-owned, so no vendor holds them hostage.
- **The context layer** — the ontology that turns the data swamp into structured, permissioned, relationship-aware objects agents can reason over.
- **Accountability** — "the moat that gets paid." Not indemnification — building the governance harness that makes the client's risk committee willing to deploy.

## Key quotes

> You thought you had a seat at the roadmap table. Turns out, you were lunch.

The strategic partner's dilemma compressed into one line: you discover, validate, and prove demand for a capability, then the vendor productises it at scale. This is the sharpest warning in the piece because it applies to the *most* collaborative-looking position.

> Go read your favorite model API's Terms & Conditions... what you promise your customers is never what your provider guarantees.

The gap between a model vendor's stochastic-output disclaimers and your own E&O policy is a structural arbitrage opportunity — and a trap. Whoever occupies that gap credibly captures the economics.

> A roadmap was never just a list of things to build. It was a set of judgments about what mattered, what could wait, and what should never be built at all. ... Now the people who make them can also ship.

The closing thesis: AI collapses the old divide between strategy (McKinsey) and execution (Accenture), so the people who make roadmap judgments can finally implement them directly.

> If you mistake technical governance for financial indemnification, enterprise legal teams will happily offload all stochastic tail-risk onto your balance sheet.

The accountability moat has a hard boundary — governance harness, not insurance underwriting — that boutique businesses cannot and should not cross.

## Themes

#concept — roadmap ownership as the axis of defensibility
#pattern — the control/context/accountability stack as the client-owned layer
#comparison — six business models ranked by moat strength

## Critical take

The essay is self-aware about its conflict of interest — the author discloses he runs a Position-6 business, and the ranking conspicuously culminates in his own model. That said, the taxonomy is genuinely useful, and the strongest insight is the T&C-versus-E&O gap: model vendors disclaim exactly the failure modes that consultancies disclaim operational responsibility for, and someone has to own that middle. His claim that VAR moats are collapsing is overstated in one direction (deep vertical integration with regulatory and data realities still resists replication) and understated in another — Position 6's "control layer" is itself under assault from the very platform vendors he warns about in Position 4. If Copilot absorbs observability and governance the way it absorbed legal workflows, the gatekeeper's second moat dissolves. The note that sticks: **accountability is the only moat that can't be absorbed by the vendor, because the vendor's disclaimers are precisely why it exists.**

A commenter adds a seventh position worth taking seriously: the independent systems integrator rolling AI out ERP-style — a shared core set up once, department-by-department rollouts justified on ROI, and model changes as routine integration work rather than existential threat. That reads as Position 6 with the churn priced in.

## Related pages

This piece sharpens [[Why Build vs Buy is the Wrong Question]] — it reframes build-vs-buy as the core *product* of the Position-6 advisor, where the client's total cost of technology, not any single decision, is the thing being sold. It complicates [[AI Killing B2B SaaS]] from the consulting side: the same commoditisation dynamics that erode SaaS also dissolve the VAR's integration IP, and platform vendors absorbing partner capabilities is the partner-ecosystem mirror of that thesis. It strengthens [[Unit Economics of AI Software]] by naming the structural gap between what model providers guarantee and what implementers must stand behind as the place where AI services margin actually lives. And its "control layer" of routing, deterministic gates, and eval harnesses is the commercial twin of the harness patterns catalogued in [[The Harness Is the Company]] — governance infrastructure as the asset, whether you sell it or own it.

---
*Sources: [[raw/who-owns-the-roadmap-six-ways-to]], [[summary/who-owns-the-roadmap-six-ways-to]]*
*Last updated: 2026-10-28*
