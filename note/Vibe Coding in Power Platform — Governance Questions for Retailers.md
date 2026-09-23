# Vibe Coding in Power Platform — Governance Questions for Retailers

Arinco's Power Platform consultant argues that AI has collapsed the cost of *creating* software but not the cost of *owning* it — so the governance barrier must move from "who is allowed to build" down to the promotion boundary, and must be enforced by the platform rather than by documents. For Nat's retail context, this piece is valuable less for its answers than for the explicit set of questions every Ontempo-built or retailer-built app should have to answer before it matters.

---

## The thesis in one paragraph

Vibe coding has arrived in Power Platform: describe an app, let an agent generate it, connect it to enterprise data, iterate conversationally. The author thinks this is genuinely good — democratisation on steroids — and simultaneously one of the biggest governance challenges the platform has faced, because the demo-to-production gap doesn't shrink with AI; it widens as the volume of builds multiplies.

---

## Key quotes

> "AI has dramatically reduced the cost of creating software. It has not reduced the cost of owning software."

The epigraph and the whole argument. Build cost approaches zero; the fourteen production questions (ownership, identity, credentials, rollback, support, who answers the pager when the creator has moved on) are untouched. This is the low-code restatement of [[Agents and Acquiring Debt]]'s comprehension debt: the debt isn't in the typing, it's in the not-understanding.

> "'But Copilot built it' is a dangerous sentence and I suspect this is going to become the new version of: 'It worked on my machine.'"

The reassignment-of-blame sentence every organisation will hear. Authorship is irrelevant to responsibility: enterprise software inherits enterprise consequences regardless of who or what wrote it.

> "Abstraction doesn't remove complexity. It relocates it and eventually somebody has to understand what was created."

The low-code insight AI accelerates: the canvas app that "is just drag and drop" sits on flows, connection references owned by individual users, custom connectors, and APIs — and the platform team finds out via "the app has stopped working."

> "Governance should increasingly become something the platform does, not something we hope people remember."

The constructive turn: a 40-page governance framework won't stop an 11 PM build, but Managed Environments, data policies, pipelines, and Solution Checker enforcement can gate at import time. Governance as deterministic check, not document — converging with [[PAAD — Defense-in-Depth for AI-Assisted Development]]'s tiered enforcement for professional agents.

> "A five-minute build can still become a five-year responsibility."

The retail-danger sentence. Every internal price-list tool or stock checker a store manager vibe-codes is a potential five-year incident.

## The questions to draw out (Nat's ask)

The article's lasting value is its question list. For Ontempo's retailers building their own tools and toys with low-code/AI, every build that starts to matter should answer:

1. **Who owns it?** — Not who built it; who is answerable when it breaks.
2. **What identity and credentials does it use?** — Personal connections are the classic low-code time bomb: the app dies when its creator leaves.
3. **What does it touch?** — Which APIs, which data, what can leave the environment, what breaks when an upstream API changes.
4. **What happens when it fails — and who finds out?** — Monitoring, incident routing, support.
5. **How do we roll it back?** — And does the support team even know the solution exists?
6. **Who changes it in six months?** — Testing, architecture documentation, and the successor's ability to understand what the AI generated.
7. **What is its blast radius?** — A personal workload organiser is not an app used by 500 staff; a departmental workflow is not an integration updating a financial system. Risk-tier, don't blanket-ban.
8. **Should this exist in production at all?** — "Knowing what should not be built" is the author's most differentiating skill.

## Analysis

Opinionated take: this is the citizen-developer governance argument rebuilt for the agent era, and its honesty is refreshing — the author doesn't blame makers, doesn't defend the status quo, and correctly identifies that "stop non-technical people building" would throw away the platform's best feature. The strongest move is the barrier-migration frame: gate the *promotion* to production, not the *act* of building, because gate-keeping at entry is both unenforceable against AI and organisationally wasteful.

The weaker spot: the piece trusts platform-enforced gates (Managed Environments, Solution Checker) to do the work, but gates only work if someone wrote the rules that feed them — which is professional developer work, quietly assumed throughout. Also, the "junior developer who never gets tired" metaphor undersells the risk: a junior's output is *reviewable by a senior*; the problem here is that nobody with the skills is in the loop at all.

## Related pages

- [[The Enterprise Gap from Vibe Coding]] — the same demo-vs-production gap seen from the architect's side of a non-technical colleague's one-day app; this source's question list is the low-code complement to its rigorix policy-enforcement answer.
- [[Agents and Acquiring Debt]] — extends the comprehension-debt framing to Power Platform's maker population, where the person LGTM-ing the output may not be able to read it at all.
- [[Platform Engineering as the AI Control Plane]] — this source's "governance becomes technical" thesis is exactly that argument applied to Microsoft's citizen-developer platform rather than to dev-tooling control planes.

---
*Sources: [[raw/vibe-coding-has-arrived-in-power-platform-and-technical-debt-has-never-been-easier-to-create]], [[summary/vibe-coding-has-arrived-in-power-platform-and-technical-debt-has-never-been-easier-to-create]]*
*Last updated: 2026-09-23*
