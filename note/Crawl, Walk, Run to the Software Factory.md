# Crawl, Walk, Run to the Software Factory

Warp's adoption roadmap for moving from scattered cloud-agent automations to a full cloud software factory: crawl with point automations, walk by deploying one complete agentic loop on a simple product, run by scaling the factory to complex codebases. Written by a vendor selling factory infrastructure, which shapes but does not invalidate the advice.

---

## The argument in one paragraph

Organisations fail to adopt agentic development not because any single automation is hard, but because point automations don't compose: they share no context, produce no cross-stage metrics, and each adds its own maintenance and security burden. The fix is a deliberate maturity path — start with trigger→agent automations (triage, review, CI healing), then deploy one complete closed loop (triage → spec → implement → review → verify → monitor) on a deliberately simple product surface, and only then scale to complex codebases where the real engineering problems live. The falsifiable claim is that this sequencing matters: teams that skip the simple-surface milestone, or that try to scale a patchwork of point tools directly, will stall — and the bottleneck at scale is factory engineering (environments, config churn, model routing, auditability), not model capability.

## Key quotes

> The thing in common with all of these approaches is that they automate a discrete part of the software lifecycle.

The crawl stage is defined by *discreteness* — and that discreteness is precisely what breaks at scale. This framing is quietly the strongest idea in the piece: the failure mode of early adoption is architectural, not competence-based.

> Note that I wouldn’t frame this as a traditional build vs. buy decision. No matter what path you take, you should expect your internal team to do some building because to make a factory approach work, that factory has to be deeply integrated into your team’s context and workflows.

A refreshingly honest reframe from a vendor who would benefit from you buying. The claim that org-specific building is unavoidable regardless of platform is testable and, so far, matches what large adopters report.

> The rule of thumb is to focus on building the pieces that are specific to your org, not the ones needed by every organization.

A clean division of labour — skills, MCPs and context on you; agent infrastructure, steering and measurement on the platform. The weak point: "needed by every organization" is exactly the layer where differentiation (and lock-in) hides.

> Getting scaled factories working is, in my opinion, going to be one of the more interesting software engineering challenges in the next few years; software engineering is becoming factory engineering.

The thesis in one line. It reframes the engineering discipline itself: the object of design is no longer the software but the system that produces it.

> The whole thing is running empirically, not on vibes.

The run-stage ideal is a factory that measures its own changes — configs, skills, model routing — the way we currently measure code. Notably, Warp admits it is "getting closer to this vision," not there.

## Critical analysis

The non-obvious contribution here is the *walk* milestone: deploy one complete loop end-to-end on a low-stakes surface (Warp used its own marketing site, automating ~75% of changes). This is better advice than it looks. Most adoption failures in the wild come from either staying in crawl forever — a dozen point automations that never compound — or jumping straight to mission-critical codebases where every automation failure is expensive. The marketing-site-as-first-factory trick buys a real closed loop at near-zero blast radius, and the 75% figure is one of the few concrete numbers offered.

The weak spots are the ones you'd expect from a vendor post. The crawl-stage failure list (no shared context, no global metrics, maintenance sprawl) is suspiciously well-tailored to what a unified platform sells. The build-vs-buy reframe is honest, but the "build what's org-specific" rule quietly assumes the commodity layer is genuinely commodity — in practice, harness choice, steering UX and observability are where factories differ most, and they're exactly what Warp wants you to rent. The run-stage bottleneck list is real but thin: "remote dev environments are hard" and "PRs pile up" are headers, not engineering.

What's left out: any discussion of what happens to engineer skill and judgement when 75% of changes ship with "minimal human touchpoints"; no failure stories or rollback discipline; no numbers on cost-per-PR or cycle time despite naming them as the metrics that matter; and nothing on the security model beyond "sandboxes are safer than local." The linked factory-stack post presumably carries that weight, but this piece asks you to accept the destination on faith.

## Related

- [[Cloud Software Factories]] — This post is effectively the adoption manual for that blueprint: Zach Lloyd's piece defines what a factory is (triage → spec → implement → review → verify → ship → monitor), and this one supplies the staged path for getting an organisation there, including the build-vs-partner split. Strongens it with an operational sequencing the blueprint lacks.
- [[Self-Improving Software Factories]] — The run-stage end state here ("agents observing the skills and config that drive the system and suggesting improvements") is the same self-improving factory thesis; this post frames it as a maturity destination rather than a design principle, and admits Warp hasn't fully reached it.
- [[Uber — Agentic Engineering Shift]] — Uber's account of adoption friction and its unclosed measurement gap is the large-scale counterpoint to this roadmap: Warp prescribes crawl-walk-run from the vendor side, Uber describes what the walk and run phases actually feel like inside a big org, including the metrics problem Warp claims its platform solves.
- [[Continuous AI]] — The crawl-stage "Trigger → Agent Activity" automations are exactly Don Syme's continuous-AI category (triage, labeling, doc updates on triggers); this post nuances that frame by arguing such automations are a transitional stage, not a destination, because they can't share context or compound.

---
*Sources: [[raw/adopting-the-software-factory-model-crawl-walk-run]], [[summary/adopting-the-software-factory-model-crawl-walk-run]]*
