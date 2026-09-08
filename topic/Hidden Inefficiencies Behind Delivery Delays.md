# Hidden Inefficiencies Behind Delivery Delays

Ahmed El-Deeb (Amazon) diagnoses why organizations keep blaming QA for delivery slowness while the real drivers — instability rework, priority churn, cross-team coupling, technical debt, and review queueing — quietly consume far more engineering capacity than testing ever did. The paper names measurable indicators for each and maps where prevailing theory (DevOps, Agile, microservices) breaks against operational reality. The thesis isn't "defend QA"; it's that optimizing what's visible rather than what's costly is the meta-inefficiency underneath all the others.

---

## Key Quotes

> "The low hanging fruit is not always the one worth cutting. Most of the time, it's the only thing we aim for just because we are too lazy or incapable of reaching out to what's deep and high."

The title thesis in one sentence. QA is the low-hanging fruit of delivery optimization — visible, timestamped, owned by a named team — and that visibility makes it the default target even when it's not the binding constraint. This is an insight that generalizes far beyond QA: any stage with clear metrics and clear owners attracts optimization attention while invisible queues silently dominate.

> "DORA has expanded its core measurement model to treat rework as first-class stability concern, explicitly connecting delivery instability to wasted effort."

DORA now tracks two rework metrics — Change Fail Rate and Deployment Rework Rate — and finds unplanned work consumes ~20% of engineering time. That's a full day per week per engineer lost to instability, not feature work. The framing shift from "how fast do we ship" to "how much do we redo" is the measurement breakthrough the industry has been missing.

> "When team is incentivized or pushed to release big batches and challenged for delivery speed, we are increasing the risk of instability work as many things will be overlooked."

Batch size pressure creates its own rework. This is the counterintuitive dynamic: the harder you push for speed, the larger the batches get, the more instability you generate, and the slower you actually go. The accelerator becomes the brake.

> "Coupling multiplies coordination, blocks independent releases, and turns local changes into cross-team programs."

A single-sentence diagnosis of why microservices didn't deliver on the autonomy promise. Conway's Law works both ways: architecture mirrors org structure, but when org structure doesn't match the architecture the system wants, every small change becomes a negotiation.

> "Meetings in that sense become friction band-aids."

Meetings aren't the disease; they're the immune response. Organizations that can't make clear decisions, document them, and trust teams to execute compensate with status syncs, alignment sessions, and CYA gatherings. Cutting meetings without fixing the coordination deficit they're compensating for just shifts the pain elsewhere.

> "Review latency (waiting time) often dominates actual review effort."

The queue itself is more expensive than the work the queue gates. Meta found P75 review time increased "by as much as a day" and Microsoft found PR dwell time 48.84% higher on "bad days." The bottleneck isn't reviewer thoroughness — it's reviewer availability and uneven load distribution.

---

## Key Themes

**#concept — Invisible queues dominate visible stages.** The five drivers (instability rework, priority churn, coupling, technical debt, review queueing) share a common property: none has a clear timestamp or owner in the delivery pipeline. They're between-stage friction, not in-stage work. The methodological breakthrough is framing each with measurable indicators — Change Fail Rate, Backlog Churn Rate, % Deployments Requiring Coordination, Time In Review (P75/P90) — so they become visible.

**#concept — Meetings as compensatory mechanism, not root cause.** Microsoft research pegs communication/meetings at ~12% of developer time, but El-Deeb argues meetings are downstream of decision debt. Poor clarity, unclear ownership, weak specs, and leadership indecision create coordination voids that meetings rush to fill. This matches the Laura Tacho finding that AI amplifies existing org health rather than fixing dysfunction.

**#concept — Theory breaks where practice bottlenecks.** The most original section catalogs four theory-vs-reality gaps: DevOps assumes smaller batches are safer (but ignores system-level interaction complexity); Agile treats responsiveness as efficiency (but ignores cognitive reset cost); microservices promise decoupled teams (but runtime dependencies survive logical boundaries); technical debt theory assumes rational trade-offs (but debt is emergent, not deliberate, and compounds invisibly). This is the paper that should be required reading before anyone deploys another microservice "for autonomy."

**#pattern — Batch size as the hidden lever.** Running through instability, coupling, and review queueing is a common thread: large batch sizes amplify every other inefficiency. They increase blast radius (instability), force cross-team coordination (coupling), and create review bottlenecks (queueing). The insight that "smaller changes = safer changes" is theoretically correct but practically insufficient — small changes in tightly coupled systems still trigger cascading coordination.

**#pattern — Delivery-only incentives are self-defeating.** When organizations reward throughput alone, teams optimize for short-term output at the expense of quality, accumulating technical debt that later kills velocity. The Meta study finding 14% of changes devoted to "code improvement" and the longitudinal study showing ~23% wasted effort from debt are the same story from two angles: pay now or pay more later.

---

## Critical Analysis

El-Deeb has written the reference article for a phenomenon everyone in engineering leadership feels but few can articulate: the things that slow delivery aren't the things we measure. The five-driver taxonomy is useful and well-evidenced, drawing on DORA, Meta internal research, Microsoft telemetry, Stripe's Developer Coefficient, and Google SRE data. The measurable indicators for each driver are actionable — a PM or EM could instrument them this quarter.

The paper's real contribution, though, is the "Where Theory Breaks" section. It's unusual to see a practitioner article that explicitly names the gaps between prevailing methodology and operational reality: DevOps' small-batch assumption, Agile's switching-cost blindness, microservices' Conway mismatch, technical debt's rational-choice model. This section alone justifies reading the paper — it's a checklist of places where dogma has outrun evidence.

The weaknesses are genuine but don't undermine the thesis. The article is structured as a taxonomy, not a causal model — we get five drivers but no analysis of how they interact. Instability rework and coupling are clearly related (tight coupling increases blast radius, which increases instability), but the paper treats them as independent. The "Opportunities for Research" section reads like a grant proposal appendix rather than a conclusion. And the title's framing around "stop blaming QA" is a rhetorical device that might read as defensive in organizations where QA genuinely is overstaffed or misconfigured.

The deepest insight is also the most uncomfortable: the meta-inefficiency is that organizations optimize what they can see. QA has timestamps and owners; instability rework has neither. The paper is arguing not just for different targets but for a different measurement philosophy — one that tracks what's invisible and costs engineering time without yielding value. That's a harder sell than "eliminate QA," which is exactly the point.

In the AI coding agent era, this paper takes on new urgency. If AI compresses implementation time but doesn't touch review latency, coupling overhead, or priority churn, then the five drivers become the binding constraints on the entire delivery pipeline. We're about to find out whether faster coding makes invisible queues more visible — or just makes them hurt more.

Priority churn in particular traces upstream to the flat backlog [[Backlog Hierarchy Problem]] diagnoses: when mixed-altitude items compete as peers, every sprint relitigates what matters, and the recency bias that drives churn is the flat list's default prioritization mode. The fix there — scoring within a level, inside a hierarchy — is the same "measure the invisible, not just the visible" principle El-Deeb applies to delivery.

---

*Sources: [[raw/el-deeb-five-inefficiencies]]*
*Last updated: 2026-07-21*
