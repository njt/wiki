# The Minimum Viable Unit of Saleable Software

Brandur (leaving Stainless to go full-time on [River](https://riverqueue.com)) works through the buy-vs-build economics in the age of LLMs. His core insight: LLMs made building cheaper but not free, and the expensive input is human oversight — the engineer who verifies, refines, and maintains what the LLM produces. That creates a "zone of viability" where software is novel enough that rebuilding is non-trivial and priced low enough that buying still beats building. The minimum viable unit of saleable software sits at the low end of that zone.

---

## Key Quotes

> "LLMs have made software considerably cheaper to build, but they haven't brought it to zero."

The thesis in one sentence. The LLM writes the code, but a human still has to read it, test it, debug it, and maintain it. That human costs ~$96/hour. The LLM eliminated typing, not engineering.

> "Anything you ship can be instantly displaced by an internal package built by an LLM."

The existential threat for SaaS founders. But the operative question isn't *can* it be displaced — it's whether the displacement is worth it. Brandur's 37-month Jira break-even says no.

> "Maintenance will be an ongoing cost."

The line every "just vibe-code a replacement" argument misses. LLM-built software doesn't maintain itself. Each Jira update, each security patch, each edge-case bug — you're paying engineer hours. The initial build is the cheap part.

> "The math here doesn't pencil out."

On the LinkedIn Jira replacement story. $400/mo is a rounding error. Even with LLMs, the build cost amortized over 37 months makes it a worse deal than just paying Atlassian. Brandur, to his credit, admits he hates Jira too — but hatred isn't economics.

> "The most expensive element being the part-time labor of the human in the equation who oversees and verifies results."

Why Brandur's framework survives even as models improve. Better LLMs reduce the refinement-loop cost but don't eliminate the human-in-the-loop. Someone has to decide whether the output is correct, and that someone bills by the hour.

---

## The Zone of Viability

Brandur's framework is a two-condition filter for software that can survive as a paid product:

1. **Sufficient novelty** — non-trivial to rebuild, even with LLMs. River's workflow engine and sequential job orchestration have enough design thought behind them that replicating them is real work, not prompt work.

2. **Moderate pricing** — not so expensive that rebuild-by-LLM becomes economically rational. Salesforce at $500/seat/mo is in danger; Jira at $400/mo total is not.

This is a surprisingly durable model. It doesn't depend on AI not getting better — it depends on *novel software being hard to specify*, which is a property of the problem domain, not the tool.

---

## Key Themes

#build-vs-buy #saas-economics #llm-economics #pricing-strategy #indie-dev #maintenance-cost #business-model

---

## Critical Analysis

Brandur is right about the economics but undersells the second-order effects — and [[Unit Economics of AI Software]] adds another: even if build costs drop to zero, *operating* costs may not. Every LLM call a product makes carries a per-user variable cost that traditional SaaS never had. A product that's trivially cheap to build can still be uneconomical to run if its inference costs eat the margin. The build/buy calculus Brandur works through assumes the operating economics are similar for both sides; AI makes that assumption false. The real threat to his framework isn't that LLMs get better at coding — it's that they get better at *specification*. If an LLM can generate a Jira clone from a three-sentence prompt and maintain it with zero human review, the entire buy-vs-build calculus collapses. We're not there yet, but the direction of travel is unambiguous.

The piece is also, transparently, an exercise in self-persuasion. Brandur is quitting his job to bet on River; the blog post is him convincing himself the math works. There's nothing wrong with that — public reasoning is how good decisions get stress-tested — but the optimistic framing (River's "sufficient novelty" as a durable moat) is doing a lot of work. Workflow engines are novel *now*. Whether they stay novel is a bet on the speed of AI progress, not a law of economics.

The strongest part is the Jira break-even math, because it's falsifiable and specific. The weakest part is the Salesforce comparison, because it assumes a 50-seat deployment and ignores the switching costs, integration complexity, and organizational inertia that keep Salesforce entrenched regardless of per-seat price. Nobody replaces Salesforce because the math pencils out; they replace Salesforce because they've decided to endure a multi-year migration.

Brandur's pricing insight — sublinear, team-based, not per-seat — is genuinely underrated. Per-seat pricing is the SaaS industry's original sin: it creates an incentive for customers to minimize usage while the vendor maximizes features. Team-based pricing (River Pro at $125/mo for up to 20 devs) aligns incentives: the vendor wins when the team grows, and the customer doesn't pay a tax on headcount. More SaaS products should price this way, and the AI-displacement threat might finally force them to.

[[The Golden Age of Open Source Applications]] complicates the framework from a third angle Brandur never considers: adoption instead of building. Graham's company doesn't rebuild Jira — it takes an existing open-source clone 80% of the way there and has an agent wire up the last 20%. That route has no build cost and no novelty requirement, so it slips entirely past the zone-of-viability filter. The cost it doesn't escape is maintenance — which lands on the open-source maintainer, not the company.

**Verdict:** A crisp, useful framework wrapped in a founder's self-talk. The zone-of-viability model is portable to any SaaS product decision. The 37-month Jira break-even is the specific number worth remembering. The open question — how fast the "sufficient novelty" bar rises as LLMs improve — is the one Brandur is betting his livelihood on.

---

## Related

- [[AI Killing B2B SaaS]] — the threat Brandur is designing around: customers rebuilding your product with LLMs
- [[The solution might be cancelling my AI subscription (Wilson)]] — Wilson's qualitative version of Brandur's math: AI output has hidden maintenance costs
- [[Inference Cost Napkin Math]] — complementary cost analysis from the infrastructure side
- [[Writing Code vs. Shipping Code]] — the gap between AI generation speed and real-world delivery: 180% gains at commit level, 30% at release
- [[Things You're Allowed to Do]] — the permission structure Brandur is operating within: quitting a job to build a paid product
- [[Software Engineering Craft]] — hub for fundamentals that LLMs accelerate but don't eliminate
- [[The Golden Age of Open Source Applications]] — the "adopt + customize" route that bypasses Brandur's build-vs-buy filter entirely

---

*Source: [brandur.org](https://brandur.org/minimum-viable-unit), May 31, 2026. Fetched 2026-06-22.*
