# The Three Pillars of Observability

Greptime's history of the "three pillars" — metrics, logs, traces — argues they were never a designed framework but three signals that evolved along separate problem domains and were grouped together only later. The bigger claim: unified storage of all three signals in one columnar store is now a solved, even commoditized problem, so the interesting question has shifted to whether agents, as first-class consumers of observability data, will force the database itself to change.

---

## Key Quotes

> "The phrase 'three pillars' rolls off the tongue so easily that it sounds like a law of nature. Nobody designed it."

The through-line of the piece. Metrics, logs, and traces answered three different questions — "is it healthy now?", "what happened?", "why did this request go wrong?" — and their engineering constraints (aggregation vs. full-text search vs. point lookups) genuinely conflict. Calling them three views of one system is a label applied in retrospect.

> "Nobody builds a cheaper Datadog anymore; they build a cheaper Honeycomb."

Charity Majors' 2024 observation on why recent observability startups converged on unified columnar storage with wide structured events. Then her 2025 reversal: "The pillar is a lie" — signal is a technical term, pillar a marketing one. Her sharper 2026 critique: vendors have the better architecture and choose not to say so; swapping the storage engine didn't change the paradigm.

> "All three signals are 'just bits,' and stacking them as three separate pillars with three separate invoices does not scale."

Ben Sigelman, co-author of Dapper and co-founder of Lightstep, at KubeCon 2018 — "Three Pillars, Zero Answers." The argument from economics: metrics blow up on cardinality, the logging bill is transaction rate × microservices × cost × retention, and treating three encodings of the same reality as three products is a pricing artifact, not an engineering necessity.

> "The destinations Bourgon had in mind were purpose-driven backends. He was describing a unified write path and a unified read model, not a requirement that every signal live in the same physical database."

A correction the article insists on. Bourgon's 2018 "über-system" is routinely claimed as a forefather of unified storage, but his proposal was more restrained: shared ingestion and access with purpose-built backends underneath. The article flags this as "frequently misread."

> "Models perform noticeably better against structured investigative primitives than against raw SQL."

The article's most consequential empirical claim. ClickHouse found a generic ClickHouse MCP server worked fine for SQL exploration, but observability investigation behaves differently from BI: wrapping log-pattern analysis, trace-outlier investigation, and cross-signal correlation into semantic tools reduced tool calls and improved consistency. An agent needs more than a connection to the data — it needs to know what the data is and how to ask.

> "Unified storage solved the data fragmentation inherited from the human era of observability. Agents raise the next problem: whether data that has been unified in storage is also unified in meaning."

The thesis distilled. Interface-layer answers (MCP, natural-language querying, Agent Skills) arrived first in 2026; architecture-layer answers — high-concurrency query capacity, unsampled full-fidelity retention, semantics as first-class metadata — are "only starting to appear, and they are nowhere near converged."

## Key Themes

- **#concept The pillars as historical accident** — three independent lineages, grouped by a marketing framework later. The "law of nature" was a constraint of early-2010s technology and budget, not a design.
- **#tool Unified columnar observability** — ClickHouse (SigNoz, ClickStack), Honeycomb's in-house store. ClickHouse is closing its own signal-specific gaps: TimeSeries, PromQL, full-text search (GA, but explicitly not BM25 relevance).
- **#person Charity Majors** — Observability 2.0 (2023): one source of truth, arbitrarily wide events, metrics/traces as derived views. The same line as Bourgon five years earlier.
- **#pattern Agent-native observability** — SigNoz, ClickStack, Grafana, Honeycomb all shipped agent access in 2026. The open question is how far down the change goes: interface, or engine and cost model.
- **#concept Semantics as the next layer** — OTel hardened semantic conventions into industry consensus; which layer actually *uses* that semantic information is still open.

## Critical Analysis

**The article declares victory and then reopens the question — and that's the honest move.** "Unified storage is solved" is stated flatly, but the entire back half is about why that victory is hollow. Storage unification is a solved problem; meaning unification is not. The piece earns its conclusion rather than asserting it.

**The ClickHouse evaluation claim carries the argument but is self-reported.** That structured investigative primitives beat raw SQL for agents is the load-bearing empirical result, and it comes from the vendor's own internal, unpublished evals. It's plausible — it rhymes with [[Text-to-SQL in the Real World]]'s finding that raw SQL generation humbles LLMs — but it's the one claim a skeptic should hold at arm's length.

**The three-pillar split was never purely technical, and the article knows it.** The table of conflicting constraints is clean, but the article is candid that business incentives — Splunk's log-search moat, three different budget pockets (SRE vs. developers vs. security) — reinforced the split as much as the engineering did. The pillars persisted partly because unifying meant incumbents giving up pricing power.

**The deepest question is deferred to a later post — correctly.** What an observability database should and should not do for a non-human consumer — whether semantics sink into the data system as first-class metadata, whether agent fan-out (dozens to hundreds of abandoned queries) reshapes query planning — is left open. That's a genuinely open architecture question, not a missing paragraph.

## Cross-References

- [[Reduce Logging Costs]] — The cost side of the same story. Observability 2.0's wide events preserve full cardinality, and the article names cost as the paradigm's most common objection — metrics stay far cheaper for aggregation.
- [[If AI Is Doing the Investigation, Version the Investigation]] — The agent-as-investigator scenario this article is building toward: read-only production access, distributed traces, and an audit trail for every query an agent fans out.
- [[The Log — Unifying Abstraction for Real-Time Data]] — Observability 2.0's "raw events as primary, pillars as derived views" is Kreps' tables/events duality applied to telemetry: the event stream is the log, metrics and traces are projections.
- [[DDB — Source-Level Interactive Debugging for Distributed Applications]] — Traces as the third pillar in action: cross-RPC backtraces and fault localization that beat GDB+OpenTelemetry — the signal that only became pressing once microservices spread.

---
*Sources: [[raw/2026-08-11-observability-three-pillars-history]], [[summary/2026-08-11-observability-three-pillars-history]]*
*Last updated: 2026-08-21*
