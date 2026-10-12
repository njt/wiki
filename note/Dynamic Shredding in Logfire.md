# Dynamic Shredding in Logfire

Pydantic's engineering write-up on making Logfire queries up to 1000x faster: instead of storing OpenTelemetry span attributes as one JSON column, automatically shred the biggest and most frequently seen keys into strongly typed Parquet columns at ingestion time — discovered per file, no configuration, capped at 128 columns.

---

Logfire stores telemetry as Parquet on object storage. The original design kept attributes as a single JSON blob and leaned on Jiter, Pydantic's fast JSON parser. The post is unusually honest about why that was a mistake: "we were optimizing to do work faster, but we couldn't really avoid doing the work." On object storage, the cost of downloading bloated JSON columns dominates CPU savings.

Static shredding (a hardcoded set like `http.status_code`) fixed it only for common keys; anything else — a `user_id`, a large `my_llm_response` — forced a full-column JSON scan, and one large rare attribute could bloat every query even when irrelevant. Dynamic shredding inverts the choice: each Parquet file gets its own physical layout, shredding the top 128 keys by total size, with type widening (Int→Float→JSON, String→JSON) and no schema churn for users, since `attributes->'key'` is transparently rewritten to the shredded column.

## Key Quotes

> "we were optimizing to do work faster, but we couldn't really avoid doing the work"

The cleanest statement of the whole problem. Faster parsing (Jiter) was optimizing the wrong axis; columnar extraction removes the work entirely. This is the classic "don't make the slow thing faster, make it unnecessary" lesson, told with real numbers (99% of files under 128 unique keys, median 32).

> "you probably have many more spans that don't involve LLM calls than ones that do, even in an agentic application"

A nice data-modelling observation with an agentic flavour: LLM-response attributes are large but rare, so unshredded they poison the shared column for everyone. Rare-but-huge values and common-but-small values need opposite treatment, and frequency-vs-size tradeoffs are exactly what the 128-key heuristic arbitrates.

> "This is the blessing and the curse of working with an upstream open source project: we get to build on a solid foundation, but we also have to wait for upstream changes to land and get released."

Six months of the project went into DataFusion itself — expression and predicate pushdown for JSON accessors, landed in DataFusion 52.1.0 — rather than into Logfire. That's the cost of building on an engine you don't own, and also the contribution strategy: fix the platform, benefit everyone, and eventually adopt the Parquet Variant type to move complexity into the standard.

> "we want to keep the logical layout of `attributes` as a single JSON column from your perspective, while optimizing the physical layout under the hood"

The user-invisible-optimization pattern, done properly this time — an explicit reaction to the earlier `_lf_attributes_*` columns leaking into `select *`.

## Key Themes

#concept #pattern #tool

- **Logical/physical split**: one logical JSON column, per-file physical shredding decided automatically — the column store equivalent of a query optimizer making layout decisions for you.
- **Bounded automation**: the 128-column cap with type widening and fallback-to-JSON for rare keys keeps the pathological case (1000 UUID keys) merely slow, never broken.
- **Safe rollout**: rewrite the query path against the old layout first, keep static shredding in the write path, flip the write path only once confident — reversible-first migration discipline.

## Analysis

The strongest thing here is the framing of the write path as a discovery problem. Shredding decisions are local to each Parquet file and derived from the data itself, which sidesteps the schema-evolution nightmare of global column decisions — a given attribute can be JSON in old files and typed in new ones, and the query rewriter bridges the gap. The type-widening rules (mixed `["123", 456]` collapses to JSON; homogeneous strings stay Utf8View) show how much careful pragmatism lives under a "1000x faster" headline.

The weak spot is that "1000x" is asserted from early-access anecdotes, not benchmarked here; the honest claim is that latency started being dominated by IO, which columnar extraction fixes. Also notable: the usage-based shredding future (shred what queries ask for, not what's big) admits the current heuristic optimizes storage, not access patterns — the right long-term answer, and the hardest one, since it couples ingestion to a query-history feedback loop.

## Relations

This strengthens [[Apache DataFusion]] with a flagship production story: DataFusion's extensibility ("the LLVM of query engines") is what made six months of upstream pushdown work possible, and the landing of that work in 52.1.0 is exactly the platform-play thesis in action. It nuances [[Postgres CDC in ClickHouse, A Year in Review]] from the other side of the data-movement world — both are "make it boring" infrastructure essays, but Logfire's answer is physical-layout discovery while PeerDB's is edge-case accumulation. It also complicates [[The Three Pillars of Observability]]'s claim that unified telemetry storage is solved: the storage format is solved, but how you lay it out per file is where the performance now lives. And it's a concrete instance of the columnar-layout tradeoffs [[Icebug Format]] navigates for graphs.

---
*Sources: [[raw/dynamic-shredding-2026-01-26]], [[summary/dynamic-shredding-2026-01-26]]*
*Last updated: 2026-10-12*
