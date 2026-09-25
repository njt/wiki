# The Year of Internal Tools

Geocodio's field report on what frontier models did to internal tooling at a ten-year, two-person company: bash scripts became full internal apps (support platform, sprint planner, papercut agent, per-branch dev environments) because AI now solves both building *and* maintaining them. The discipline that makes it work is almost entirely pre-AI craft — weeks of spec-writing, adversarial plan interrogation, shared UI infrastructure, docs kept in lockstep with code — with deliberate limits on what stays off-the-shelf.

---

The economics argument is the interesting move. The authors' pre-AI blocker was never construction — "Even pre-AI you might be able to knock out some kind of awesome internal tool over a couple of days" — it was that each tool accrued a maintenance obligation you had to budget against forever. Their claim is that AI collapsed *both* terms, which is a stronger position than most build-with-AI essays take.

> "I am not worried about building all these internal tools and maintaining them, because I can use AI to keep up with the maintenance as well."

That confidence is doing a lot of work, and the article itself half-undercuts it: "Some of the tools exist to keep the other tools healthy." Tool populations that spawn their own caretaking tools are exactly how a two-person company gets eaten by its own estate. The Willison position in [[The solution might be cancelling my AI subscription (Willison)]] — that AI generates maintenance obligations faster than you can meet them — is the direct counterpoint; Geocodio is the optimistic rebuttal, but only because they deliberately capped the population.

> "The planning and the spec take weeks. The implementation takes hours or days."

The inversion of the old ratio, stated cleanly. Their pipeline: significant planning, engineering spikes, repeated "Grill Me sessions" using Matt Pocock's skill "which interrogates a plan until the weak parts fall out," and UI mockups that "answer a surprising number of questions" before a line of real code. This strengthens [[Matt Pocock — Grill Me, Then Go AFK]] with an independent production endorsement, and reinforces the spec-as-artifact economics of [[SDD Case Study — 13 Apps in 70 Days]] — note their observation that Atlas "had been in my head for years, and once the spec existed, building it was the fast part": for some projects the spec is mostly *harvesting* accumulated understanding.

> "A staging environment for every internal tool is one more thing to keep running... Changes go to production, and local QA, test coverage and CI carry the weight that staging would otherwise carry."

The most honest paragraph in the piece. No staging for internal tools is a deliberate trade-off, and they got a day-long ETL outage as the receipt: "The ETL outage is what that decision looks like when it goes wrong, and we accepted that when we made it." Contrast with [[Ship Safe]], which treats that kind of risk transfer as a decision worth surfacing explicitly.

## Key themes

#pattern — every tool ships a documentation subsite, with a CLAUDE.md rule ("Changed behavior → edit the relevant page(s)... Don't leave stale... claims") and a DocsManifest as single source of truth. Docs-as-manifest is the interesting concrete artifact here: documentation becomes a validated, registered surface rather than a README that rots.

#pattern — build-vs-buy as three questions: domain experience, cost of a day down, and whether the value comes from integration with your own data. Email stays at Bento, Twilio keeps comms, "Nobody here is writing a Slack." A crisp internal-tools variant of the buy-vs-build zone of viability in [[The Minimum Viable Unit of Saleable Software]].

#tool — the fleet itself: Atlas (support), Bullpen (sprints), Yak (open-source papercut agent picking up tasks from Slack/Linear/Sentry/GitHub and returning PRs), Treehouse (per-branch worktree + docker-compose stack, also open source — an alternative take on the isolation problem [[Domenic Denicola's Agentic Coding Setup]] solves with disposable VMs).

#concept — shared infrastructure as the anti-sprawl mechanism: one console-ui package, shared auth standards, shared CI, and an explicit rejection of a "micro-internal-tool-services architecture." Tools cover whole feature sets rather than one new service per idea.

## Opinion

The article's real contribution is naming maintenance as the *first-order* variable in internal tooling, not a footnote — and then being candid about the residual risks (no staging, VPN-gated access, an outage they ate on purpose). It is less candid about the model-dependence of the maintenance claim: "AI keeps up with maintenance" is asserted, not measured, and the whole strategy compounds risk on it. Still, against the sprawl dystopia of fifty half-loved tools, their mitigations — shared UI, deliberate caps, "start small, close the gap you already work around by hand" — are the most concrete version of this thesis I've seen from a real two-person shop.

---
*Sources: [[raw/2026-09-23-the-year-of-internal-tools]], [[summary/2026-09-23-the-year-of-internal-tools]]*
*Last updated: 2026-09-25*
