# Reduce Logging Costs

Michael Shpilt's practical taxonomy of five strategies for cutting observability spend, from sampling and tiered storage through vendor migration to the nuclear option of killing INFO logs entirely. The core argument: sustainable cost reduction comes from reducing unnecessary telemetry before it leaves your application — treating logs as code requiring continuous maintenance rather than a fire-and-forget exhaust pipe.

---

## Key Quotes

> Logging costs can reach up to 5 or 10 (or more) percent of your cloud host costs. That's just too much.

The opening salvo. Whether 5–10% is actually "too much" depends on what those logs are buying you — the article never fully grapples with the value side of the cost/value equation. But the number is directionally correct and the instinct is sound: if most logs are never queried, you're paying to store landfill.

> The events you care about most are often the rare ones — a production bug occurring once in 10,000 requests, unusual user flows that are important but infrequent, and one-time failures.

This is the killer argument against naive sampling. Sampling is cheap and easy, but it's a uniform filter on a non-uniform signal. The most valuable logs are exactly the ones sampling is most likely to discard. If your sampling strategy doesn't account for signal rarity, you're optimizing for cost at the expense of the very incidents that justify having logs at all.

> Developers want to add features and work on the codebase, not to do house cleaning.

The structural reason log rot accumulates. No one's performance review says "reduced log volume by 40%." Log hygiene is a public good that no individual team is incentivized to provide. This is a coordination problem masquerading as a technical one — and it's why Shpilt's own tool (Obics) exists as a business.

> A radical move for a radical situation. On the upside, you'll definitely save on your observability costs. On the downside, you're almost completely blind in production.

The INFO-log removal strategy gets exactly the framing it deserves: it's not a strategy, it's a surrender. Raising the floor to WARN means you can't answer "what happened?" for anything that didn't trigger an explicit warning. It's the observability equivalent of turning off the lights to save on electricity.

> Reduce unnecessary telemetry before it ever leaves your application.

The thesis sentence. Everything else — sampling, tiering, vendor-switching — is downstream treatment. The only strategy that saves CPU cycles, network bandwidth, storage, *and* future engineering time is not generating the data in the first place.

---

## Key Themes

- **#concept Telemetry as code** — Logs aren't an exhaust byproduct; they're code that produces data instead of behavior. Like code, they accumulate cruft, need refactoring, and benefit from review. The article's most important reframe.
- **#pattern In-code optimization over infrastructure optimization** — Fixing logging at the source (deduplication, aggregation, level reduction) beats downstream bandaids (sampling, tiering, vendor migration). Same insight as [[Queues Don't Fix Overload]]: treat causes, not symptoms.
- **#tool Observability vendor economics** — The vendor lock-in dynamics are real: DataDog charges for ingestion even when you exclusion-filter the logs, tiered storage makes cross-tier queries "a nuisance," and migration costs include retraining engineers on a new tool. The switching cost is the moat.
- **#pattern The observability cost ladder** — The five strategies form an implicit ladder from least destructive (in-code cleanup) to most destructive (killing INFO logs). The further down you go, the more you trade visibility for savings. Most teams start from the wrong end.

---

## Critical Analysis

**The article has the right thesis but the wrong ordering.** Shpilt leads with sampling and tiered storage — the vendor-side solutions — and saves in-code cleanup for strategy #3. The argument the article actually makes is that in-code cleanup is the *only* strategy that reduces costs without degrading signal quality. That should be strategy #1, with sampling and tiering presented as supplements for the remaining cost, not as primary approaches.

**The Obics pitch is transparent but not disqualifying.** The article originated on the Obics blog and the tool gets a dedicated mention. But the tool solves a real problem (identifying redundant logs programmatically) and the article's advice doesn't depend on using it. The disclosure is there; the analysis stands on its own.

**The article underplays the organizational problem.** "Developers don't enjoy cleanup sprints" is as close as it gets to naming the incentive structure that creates log rot. But the real question is: who owns logging costs? In most organizations, the team generating the logs doesn't see the Datadog bill. Until that feedback loop closes, no amount of tooling or good intentions will prevent re-accumulation. This is the same dynamic [[Patreon Notification Fanout]] diagnosed in platform migrations: the work that matters isn't the work anyone's roadmap rewards.

**The re-accumulation isn't just organizational — it's the vendor's business model.** [[The Observability Pain Cycle]] names the structural loop Shpilt's tactics can only interrupt, never end: observability vendors profit from store-everything-up-front, so trimming is triggered by invoice pain, applied coarsely, and never re-tuned — which is why volume silently grows back and the cycle repeats. Shpilt's in-code cleanup treats the symptom; Petkovic's diagnosis explains why it keeps recurring.

**What's missing: structured logging as the foundation.** The article treats all logs as a uniform blob to be reduced. But structured logging (JSON key-value pairs, consistent schemas) changes the economics: it enables cheaper storage (columnar compression), faster querying, and automated analysis that can identify redundancy programmatically. If you're going to treat telemetry as code, structured logging is your type system.

**The connection to agentic development is unexplored but real.** AI coding agents generate logs at scale — they instrument code, add debug statements, and rarely clean up after themselves. As agent-generated code becomes a larger share of production systems, the log hygiene problem compounds. The article's "autonomous cleanup" future with Obics is actually a special case of a broader pattern: agents generating telemetry and agents cleaning it up, with humans at the review boundary.

---

## Cross-References

- [[The Future of Software Engineering is SRE]] — The article that establishes observability and operations as the scarce skill in an AI-coding world. Shpilt's strategies are the cost side of that equation.
- [[Software Engineering Craft]] — The hub page covering operations, simplicity, and the judgment calls that matter more when code generation is cheap.
- [[Queues Don't Fix Overload]] — The same structural insight applied to a different domain: downstream bandaids (sampling, tiering) treat symptoms; fixing at the source (in-code cleanup) treats causes.
- [[Anomaly Detection]] — A concrete example of operational tooling done right: Welford's algorithm, no ML, no config. The kind of targeted signal extraction that makes log volume reduction safe rather than reckless.
- [[Patreon Notification Fanout]] — Shares the diagnosis that observability/logging data models aren't afterthoughts — they're the feature that makes platforms maintainable. Also shares the organizational bottleneck insight.
- [[Lean, Not Backpressure]] — The lean manufacturing lens: fixing quality at the source rather than inspecting and filtering downstream. Shpilt's "reduce before it leaves the application" is jidoka for telemetry.
- [[The Three Pillars of Observability]] — The broader history the cost ladder sits inside: metrics, logs, and traces evolved separately and are now unified in one columnar store. Its cost objection — that wide events preserve full cardinality, keeping metrics cheaper for aggregation — is Shpilt's dilemma seen from the vendor side.

---
*Sources: [[raw/reduce-logging-costs]], [[summary/reduce-logging-costs]]*
*Last updated: 2026-07-18*
