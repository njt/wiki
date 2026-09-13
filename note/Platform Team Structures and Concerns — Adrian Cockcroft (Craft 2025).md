# Platform Team Structures and Concerns — Adrian Cockcroft (Craft 2025)

Adrian Cockcroft — the cloud architect behind Netflix's AWS migration, later an Amazon VP, now advising Nubank — uses his Craft 2025 keynote to argue that platform engineering is an incentives problem wearing an architecture costume: platforms are layered stacks that sediment functionality upward, vendors are structurally incentivized to fatten the layers they sell you, and the machinery that lets a platform be simultaneously stable and fast-changing is version-aware traffic routing built at Netflix in 2010 and still underused today.

---

## Key quotes

> "If somebody says, well, the problem we're having to solve is installing Kubernetes, that's the wrong answer."

The anchor of his Wardley-mapping section: platform work must trace to a user or business need — faster delivery, international launch, lower cost — never to the technology. Coming from the person whose teams did adopt Kubernetes-adjacent infrastructure at scale, this lands harder than a vendor-neutral consultant saying it.

> "The incentive of the vendor is to fatten their layer so they can charge you more for it... which is good at some times, but it's also the wrong incentive if you're trying to build a platform that's thin and evolves rapidly."

The core argument for open source over vendor platforms, and the reason Netflix contracted vendors to support open source tools rather than buy fat platforms. This is also the quiet theory behind the whole thin-layer-on-stable-base strategy.

> "There is no perfect way to do versioning. There's just some less bad options."

The Netflix conclusion after huge internal debates. Refreshing against the standard conference format of prescribing one true scheme — every option has downsides, pick deliberately.

> "Running core banking on Kubernetes on the spot market on ARM chips, which is, if you're a traditional banker, your head explodes somewhere in that sentence."

Nubank's stack (118M customers, ~1,000 deploys/day, Datomic on DynamoDB) delivered as a dare to conventional wisdom about what banking infrastructure must look like.

> "If you want things to look tidy, then you will find it is very hard to evolve."

His defense of running many incompatible service versions simultaneously — "It looked confusing, but it works." The counterintuitive claim is that tidiness is an aesthetic preference that actively slows evolution.

> "You can replace me with a shell script that says, we did that at Netflix 15 years ago."

Why he built an AI persona (Supra) trained on his own published answers — most questions he gets are ones he has already answered. He judges the persona's output a better answer than he'd bother typing in email.

> "Everyone's heard of observability at this point. Controllability... is the next thing. You can observe what's going on. Can you control your system? If you can control your system, it's reliable."

His framing for chaos testing, game days, and incident management as one discipline — the reliability team under a new name.

## Key themes

- **#concept** — Layered platform stacks and sedimentation: functionality accumulates upward, commoditized pieces sink downward; platforms must stay thin to evolve.
- **#concept** — Internal vs external platform physics: externalized interfaces must be stable for years (TV firmware), internal ones should change constantly and messily.
- **#pattern** — Version-aware traffic routing as the fundamental primitive: immutable additive deploys are zero-risk until traffic routes; stability then "sediments across" from change-hungry teams to everyone else.
- **#pattern** — Facets: client, server, and cache hold different object models so shared definitions can't lock microservice versions together.
- **#tool** — Wardley maps (and MapKeep) as the team communication artifact for platform migration; Doomsday Clock prioritization (Kat Swetel, Nubank); Megpt + Supra for an AI persona from published content; Cursor + Claude for vibe-coding internal tools.
- **#person** — Adrian Cockcroft, with a supporting cast of Netflix/Nubank veterans: Kat Swetel, Michael Nygard, Rich Hickey, Stu Halloway.

## Critical analysis

The title over-promises and the talk is better for it. "Team structures" gets one sentence of Team Topologies endorsement; what Cockcroft actually delivers is platform *architecture* plus an incentives argument, and the incentives argument is the freshest part. The internal/external physics split — externalized interfaces must fossilize while internal ones must churn — is the cleanest one-slide explanation of why vendor platforms disappoint that this wiki has seen: it reframes build-vs-buy from a cost question into a rate-of-change question.

The versioning section is genuinely contrarian against current industry defaults. Mainstream practice converged on replace-in-place rolling deploys plus GitOps, where rollback means redeploy; Netflix's 2010 design treats rollback as *routing* — every push lands as a new additive autoscale group running dead alongside the old one, and control is exercised by shifting traffic, not by undoing code. His bewilderment that "a lot of people still aren't using it" is half fair, half self-inflicted blindness: service meshes like Istio now do traffic shifting natively, and Nubank — the case study in his own talk — runs exactly that, a tension the gist's digest rightly flags as unexamined. The honest disclaimer ("I left Netflix in 2014, so what they do now, I don't know") is a feature, not a bug; it marks the material as primary-source history rather than current best practice.

The most transferable artifact is not the versioning machinery but the Doomsday Clock: for every platform component, the product manager must estimate when it stops working and how long the fix takes, then sort the portfolio by that and clear the top. It is a forecasting discipline, not a backlog heuristic, and it sidesteps the roadmap-versus-tech-debt fight by making time-to-failure the unit of comparison. Its weak point is the one the digest names: the sorting is described, the estimating is glossed.

The gaps are real and the digest enumerates them well: no adoption story (the classic platform-engineering failure, build-it-and-they-don't-come, is absent — no developer-experience metrics, no KPIs anywhere), no legacy-migration path for the data-center-bound majority of the audience, Nubank's Brazilian banking regulation reduced to a punchline, and the facets pattern's maintenance cost (who keeps three object models in sync?) unexamined. The AI persona segment is a tangent that crowds out the promised "how to make it work" material — but as a data point it is instructive: a retired luminary whose FAQ workload is fully automatable by his own published corpus is the mildest possible demonstration of the replacement argument, delivered with a laugh.

## Related pages

- [[From Here to There and Everywhere — Simon Wardley, Craft 2025]] — Cockcroft's middle section is the worked practitioner application of Wardley's own Craft 2025 morning keynote: user need as the anchor, the evolution axis, and team argument over every buy-vs-build arrow on a real travel-search map. Where Wardley argues the method, Cockcroft shows the org meeting where the map becomes the roadmap.
- [[Who Does What — Team Topologies for the Agentic Platform]] — Cockcroft takes the platform-team concept straight from Team Topologies and adds what Wulveryck's agentic extension lacks: the funding and incentive machinery (explicitly hired platform PMs, vendor fattening, open-source contracting) that determines whether the platform actually stays thin.
- [[The Rise and Fall of eBay Velocity]] — the sibling Craft 2025 case study. eBay's transformation succeeded at the delivery layer and still couldn't move the business; Cockcroft's Netflix/Nubank stories are platform architecture that did move the business. Read together, they show org failure and platform failure operating in different registers — and both speakers bypass vendor platforms in favor of thin in-house layers.
- [[In Praise of Normal Engineers]] — Majors prescribes platform engineering for humans (short deploy intervals, engineers own their code in production); Cockcroft supplies the organizational mechanism that makes such a platform a product rather than an internal charity, with a hired PM owning the roadmap.

---
*Sources: [[raw/16a182951174292648870ac987f65415]], [[summary/16a182951174292648870ac987f65415]]*
*Last updated: 2026-09-13*
