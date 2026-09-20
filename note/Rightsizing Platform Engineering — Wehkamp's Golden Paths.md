# Rightsizing Platform Engineering — Wehkamp's Golden Paths

A decade-long field report from Wehkamp, one of the Netherlands' largest online department stores, arguing that the hardest problem in platform engineering is not building the platform but deciding how much platform to build. Written by the team that lived it, expanding their KubeCon EU 2026 talk "How Much Platform Is Enough Platform."

---

## The argument in one paragraph

Platform engineering fails most often by over-building: the comprehensive internal developer platform that supports every use case is a maintenance liability, not an asset, and the successful platform is the smallest one that removes the organisation's actual delivery bottlenecks. Wehkamp claims this from experience — they tried a full self-serve portal with strict RBAC and abandoned it, tried adopting Backstage multiple times and abandoned it, and converged instead on a handful of opinionated golden paths, an "apply or explain" escape hatch, and a governance model that sorts resources into consume-only versus multiparty. If the claim is right, the dominant failure mode of platform teams is scope, not execution — and most platform roadmaps in the industry are pointed at the wrong target.

---

## Key quotes

> "solving for one kind of toil shifts focus to the next kind of toil, just like solving for a bottleneck in a complex system realistically just makes something else the new bottleneck"

The article's best idea, and the reason "done" is the wrong goal for a platform team. Every platform win manufactures the next bottleneck — here, going from quarterly to weekly releases turned resource provisioning into the new constraint at a hundred releases a week.

> "Choose the simplest platform that solves your organization's problems; successful platform engineering is measured by improved delivery and reduced cognitive load, not by the number of platform features."

The thesis in one line, and a direct rebuke to platform teams that benchmark themselves against Backstage screenshots from Spotify's keynote. Note the metric is cognitive load and delivery, not adoption or feature parity.

> "Realistically, that design doesn't alleviate any cognitive load."

Said of their own first portal design — a comprehensive self-serve system with complex quality controls and RBAC matrices so teams could manage every technical detail themselves. This is the article's most honest sentence: the design that looks most empowering on a slide is often just the old handoff problem with extra steps.

> "We tried to implement Backstage multiple times, not realizing that the upkeep looked a lot like our previous ops-driven sprints, in which a successful outcome would be dependent on other teams collaborating and bringing their own additions and maintenance capacity."

A rare admission about the industry's most-cited platform project. Backstage failed at Wehkamp not on features but on the same dependency structure that made their pre-platform org dysfunctional — success required contributions and maintenance the consuming teams never had capacity to give.

> "This pattern makes service exposure with write operations an explicit choice rather than an accident."

The strongest concrete example of what good platform defaults look like: services are not exposed to traffic by default, WAF and mTLS cannot be skipped, and enabling unsafe HTTP verbs is a deliberate act. Security posture emerges from the golden path rather than from per-team diligence.

---

## Critical analysis

The non-obvious contribution here is the two-axis governance model. Most platform engineering writing stops at "golden paths good, portals hard." Wehkamp's split of resources into **consume-only** (cloud infrastructure, observability — you build with it, never define it) versus **multiparty** (messaging, traffic — everyone must follow the same rules or it breaks) is a genuinely usable classification, and their observation that Kafka spans both — multiparty topics on consume-only brokers — shows the model has teeth. The "crescendo" of divergence is equally practical: staying on the golden path is nearly free, and every step off it demands more collaboration, budget, and proof of need. That is a pricing mechanism for complexity, which is exactly what platform governance usually lacks.

The weaknesses are what the article smooths over. First, the narrative is suspiciously linear for a story that explicitly says the changes "didn't happen linearly" — a decade of iteration is compressed into a tidy arc, and the costs of the abandoned portal and Backstage attempts (years? headcount? morale?) go unquantified. A reader cannot tell whether rightsizing was insight or exhaustion. Second, the organisational preconditions are underexamined: the platform team got real buy-in because "everyone had already seen and experienced the benefits" of the first transformation. Most organisations attempting this have no such shared success to trade on, and the article offers nothing for them. Third, the "apply or explain" rule is described but never tested — we never see a team actually explain their way off the golden path, which is precisely where such policies tend to rot into either rubber-stamping or silent shadow infrastructure.

What is left out: any engagement with the AI-agent layer. Written in 2026, the article treats the platform's users as human engineers picking from a GUI menu of building blocks. But if coding agents are becoming the primary consumers of platform surfaces — provisioning, deploying, configuring — then the golden-path economics change: agents do not experience cognitive load the same way, they iterate at machine speed against provisioning latency, and the "apply or explain" gate becomes an authorisation question. The article is a strong document about the pre-agent platform era that does not notice the era ending.

---

## Related

- [[Platform Engineering as the AI Control Plane]] — that source argues platform engineering is absorbing AI toolchain ownership and becoming the control plane for agents; this Wehkamp report strengthens its foundation by showing what platform engineering looks like when done well for humans, and complicates it by suggesting the golden-path model was designed around human cognitive load that agents may not share.
- [[Platform Engineering End-to-End]] — both cover the full arc of building an internal platform, but this source nuances that picture with a concrete failure case (three abandoned Backstage attempts) and a governance taxonomy (consume-only vs multiparty) that the end-to-end view lacks.
- [[Platform Team Structures and Concerns — Adrian Cockcroft (Craft 2025)]] — Cockcroft addresses how platform teams should be organised; this source supplies the demand-side complement, arguing that the platform's scope should be set by delivery bottlenecks and team capacity rather than by the team's own ambitions.
- [[Stevey's Google Platforms Rant]] — the rant demands platforms be like products with a "head of the platform" accountable to users; a decade later, Wehkamp's "platform as a product" practice — retiring features, responding to shifting ticket patterns, treating adoption as the signal — reads as the empirical vindication of that demand.

---
*Sources: [[raw/rightsizing-platform-engineering]], [[summary/rightsizing-platform-engineering]]*
