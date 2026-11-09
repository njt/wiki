# The Palantir Model (Scaman)

Zoe Scaman extracts the business architecture underneath a company she detests: Palantir embeds its people inside client organisations as residents, assumes the client is wrong about its own problems, diagnoses from within, and generalises what works into a compounding pattern library. She argues this — not the software — is the product, and that strategy (in advertising especially) must migrate from the agency conveyor belt to embedded organisational intelligence.

---

Scaman opens with a disclaimer worth preserving: "you can admire the blueprint while despising the builder." Her target is the *model*, not the morals — and the model is simple to state. Most enterprise software (and every agency) sells off-the-shelf output through a conveyor belt: brief in, campaign out, eight to twelve weeks. Palantir instead collapses "discovery, diagnosis, design, build, embed, iterate" into one long residency, sitting "inside the mess until they understand it better than the people who created it."

The key structural insight is the two-tier engineering split:

> Forward Deployed Engineers go into client organisations and build fast, hacky, whatever-works solutions... Then there's a second group — Product Development engineers — whose job is to take those bespoke solutions and generalise them. Find the patterns. Build reusable infrastructure. Turn the one-off into the scalable.

This is "the encoding mechanism" — how knowing becomes substrate. And it is exactly what the indie embedded-strategist wave lacks: they do the FDE job with no PD layer behind them, so "every engagement is linear, not exponential... a very experienced person who can only be in one room at a time."

The second thread is about access, not analysis. Palantir's core unglamorous engine is data integration, and data integration is political: "a team controls a data source. Their entire existence depends on being the gatekeepers." Hence *Impro* — the FDE job is infiltration for the client's benefit: read status dynamics, become wallpaper, earn the truth nobody puts in a brief. Scaman, quoting Tyler Cowen's "context is that which is scarce," lands on the difference between "tell me your problem" and "let me watch you work for six months" — stated versus revealed preferences.

She is honest about the dark side: embedding at this depth is "dependency by design," load-bearing lock-in. And it demands clients desperate enough to let strangers see the sausage being made — a precondition most organisations haven't met yet.

**Key themes:** #concept (embedding as business model, FDE/PD split) #pattern (flywheel from bespoke to productised) #person (Tyler Cowen's scarcity of context; Keith Johnstone's *Impro*) #tool (the missing "substrate" for encoding indie practitioners' knowledge)

## Analysis

The essay is strongest where it's most uncomfortable. The agency critique is almost a throwaway — in-house agencies as "the same model, different postcode" is a good line but the real point is the billing impossibility: "You can't bill for six months of watching. You can't scope 'figure out what's actually broken.'" That's a genuine institutional analysis — agencies can't sell diagnosis because SOW economics select against it, not because strategists lack the skill. The value isn't in the output; it's "the understanding that makes the right output possible."

The flywheel argument is the most transferable idea, and it maps directly onto debates in the AI engineering world about what actually compounds: not code, not models, but the accumulated pattern library of engagements. The unanswered question — "how do we encode understanding without flattening it into templates?" — is the same problem every knowledge-management system has ever had, and she doesn't pretend to have solved it.

Where the essay is weakest: "the product is embedded cognitive capacity" flirts with the consultant-speak it disclaims, and the essay never seriously engages with why anyone should trust a model whose reference implementation is a surveillance company that benefits from exactly the lock-in Scaman flags. Her proposed corrective — "capability transfer as an explicit goal, not just a nice-to-have" — is a one-line constraint against a structural incentive, which is optimistic. But naming the trade-off at all is more than most business-model writing manages.

## Relation to the wiki

- [[Who Owns the Roadmap — Six Ways to Position an FDE Business]] — this essay supplies the cultural and political anthropology behind that taxonomy's "Palantir position": van den Veld maps the business models of FDE work; Scaman explains why the position works — residency, distrust of briefs, the FDE/PD encoding loop — and what it costs in lock-in.
- [[AI, Tools and Transformation]] — compels the same distinction from the other side: Evans argues transformation fails because organisations can't articulate what they need and tools alone don't fix it; Scaman's embedded-residency model is precisely the "knowing you need one" problem Evans identifies, answered with an institutional design rather than a product.
- [[The Harness Is the Company]] — strengthens the compounding claim: Shankar argues the business that owns its outer loop owns the value; Scaman's FDE/PD split is a human-era instance of the same encoding mechanism, with the open substrate question now being whether agents can play the PD role.

---
*Sources: [[raw/the-palantir-model]], [[summary/the-palantir-model]]*
*Last updated: 2026-11-09*
