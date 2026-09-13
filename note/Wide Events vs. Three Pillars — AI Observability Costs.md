# Wide Events vs. Three Pillars — AI Observability Costs

Honeycomb's Nick Travaglini argues that AI agents make observability costs balloon not because agents emit more telemetry, but because the three-pillar model (metrics, logs, traces as distinct formats in distinct stores) makes you pay for the same fact three times — and that the wide event model, storing one rich structured event from which metrics, logs, and traces are all derived, is what keeps AI observability spend predictable. The piece's genuinely new claim is about retention: non-deterministic agents produce statistical output, so the classic cost lever of cutting retention backfires precisely when you need long windows to see behavioral patterns.

---

## The Argument

Agentic AI adds telemetry dimensions that conventional systems never had: which model ran, which skills were invoked, whether each tool call worked, what failovers the model "thought" to try, the initiating prompt. Under the pillar model these get recorded redundantly — a unique prompt becomes a one-off datapoint in a dedicated time series, the prompt text lands in a log, and the conversation's start opens a trace — in three stores, with a correlation effort on top. The cost correction: retention isn't the dominant expense, ingest and query compute are, and AI's statistical nature demands *more* history, not less.

The alternative is the wide event: one structured file (Honeycomb uses flat key:value pairs, no nesting), as wide as the ingesting system can practically take, from which everything else is a derived view — durations recorded directly, p95 computed at read time, traces composed through parent identifiers, and agent-and-sub-agent conversations composed into an "Agent Timeline." Store once, pay once, derive forever.

## Key Quotes

> "Here's the honest truth: AI is software. Unpredictable, but software nonetheless."

The framing move that keeps this an observability argument rather than a new discipline. The system's behavior stays unpredictable; the point is that the *bill* doesn't have to be. Predictability of spend substitutes for predictability of output.

> "That's not even considering the headache of the additional overhead of trying to reconcile them, which isn't possible because the distinction is required by hypothesis."

The sharpest sentence in the piece. Under the pillar hypothesis, three records of the same fact cannot be reconciled — reconciliation would erase the distinction that justified the split in the first place. The correlation work is not an implementation failure; it is structurally mandated.

> "Since AI agents are non-deterministic and therefore outputs are statistical, organizations are going to want long-term data so they can suss out behavioral patterns that only emerge with large numbers."

The piece's most original cost claim. Every existing observability-cost playbook assumes deterministic systems where old telemetry stops teaching you anything; agents invert that assumption, and with it the retention-cutting reflex.

> "Undoubtedly, the three pillars model, especially when metrics are first among equals, can be cheaper in some cases due to preaggregation. However, it's the inherent uncertainty of whether your case is one of those 'some cases' that's the problem."

The honest concession — and, on inspection, a rhetorical one. It repositions the debate from "which is cheaper" to "which is *predictably* priced," which is the frame wide events win by construction. But no worked example of a pillar-favorable case is ever given; "some cases" carries the entire counter-argument.

## Key Themes

- **#concept Wide events / Observability 2.0** — one arbitrarily-wide structured event as the stored primitive; metrics, logs, and traces as read-time projections. The definition is inherited wholesale from [[Charity Majors]]' 2.0 post, quoted at length.
- **#pattern Store once, derive at read time** — p95 on the fly, traces from parent IDs, Agent Timelines from agent-and-sub-agent events. Aggregation at read preserves raw events for ad hoc querying.
- **#concept Cost predictability as an architecture property** — the claim that uncertainty, not absolute cost, is the thing to engineer away; simplicity of the data model is what makes spend forecastable.
- **#person Charity Majors** — the canonical definition the piece builds on; Travaglini's contribution is applying it to the AI-agent cost problem, not extending the model.

## Critical Analysis

**This is a vendor post with a predetermined conclusion.** Honeycomb's architecture is the thesis; the closing paragraph says so ("consider using Honeycomb"). That doesn't make the argument wrong, but it explains what the post never does: engage a worked example where pillars genuinely win, or ask what wide events cost at read time.

**The read-time compute tension is the post's unexamined contradiction.** It correctly identifies ingest and query compute as the dominant costs — then advocates a model that maximizes read-time compute (GROUP BY over raw events, p95 computed on the fly). The reason that's affordable in practice is Honeycomb's high-cardinality columnar engine, not the data model itself. [[The Three Pillars of Observability]] records the neutral version of this: unified columnar storage is now commoditized, so the differentiator is the engine underneath, and vendors with the better architecture "choose not to say so." This post is an instance of the phenomenon it documents.

**Flat, no-nesting events versus nested agent execution.** Agent conversations are naturally trees — agent → sub-agent → tool call → retry. The post waves this off with "the only limit... is the practical limit of the system ingesting the file" plus the parent concept, but never shows what reconstructing a deeply nested conversation from flat events looks like, or what an Agent Timeline costs to build. The most AI-relevant construct in the piece is asserted, not demonstrated.

**The retention inversion is the durable contribution.** Cutting retention is a mainstay move in [[Observability Cost Saving Strategies]] and [[Reduce Logging Costs]] — sampling, tiering, trimming. For statistical, non-deterministic workloads that lever turns harmful: you need the long tail to see the behavior pattern. That single point justifies the post's existence independent of the wide-event sales pitch.

**The checklist is the practical takeaway.** Separate request-level from derivable data, model cardinality/retention/growth for forecasting, and — the genuinely AI-native move — "connect token and model activity to telemetry volume." That last one makes observability spend a function of inference spend: two meters, moving together, that no earlier cost piece in this wiki treats as one system.

## Cross-References

- [[The Three Pillars of Observability]] — Strengthens the same argument from the cost side: Greptime's history ends asking whether agents as first-class telemetry consumers force the observability database to change; this is a vendor answering with the cost dimension. It also nuances: Greptime treats the pillar split as a historical accident of separate lineages, while Honeycomb treats it as a pricing artifact — the same conclusion ("three invoices for one reality") reached from archaeology and from economics.
- [[Observability Cost Saving Strategies]] — Complicates it: Shpilt's seven strategies all optimize *within* the pillar model (logs-to-metrics conversion is the pillar split's economics in action), while this post claims the model itself is the cost driver — and its non-determinism argument inverts the playbook's retention-tiering reflex for AI workloads.
- [[The Observability Pain Cycle]] — Extends it: Petkovic's boom-bust loop (ingest everything, invoice shock, coarse trim, silent regrow) is a disease of the pillar model's duplicated storage; wide events claim to remove the duplication that fuels the regrowth — though the post never asks why earlier "one platform" consolidation promises failed to break the same cycle.
- [[If AI Is Doing the Investigation, Version the Investigation]] — Complements it on the write side: that piece asks how agents *read* telemetry (versioned investigations, audit trails); this piece asks what gets *written* and what it costs. The Agent Timeline is the artifact both need — asserted here as a product feature, examined there as an accountability problem.

---
*Sources: [[raw/wide-events-vs-three-pillars-ai-observability-costs]], [[summary/wide-events-vs-three-pillars-ai-observability-costs]]*
*Last updated: 2026-09-13*
