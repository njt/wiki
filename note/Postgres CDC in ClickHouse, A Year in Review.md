# Postgres CDC in ClickHouse, A Year in Review

Sai Srirampur (PeerDB founder, now at ClickHouse) reflects on the first year after acquisition: PeerDB became the engine behind ClickPipes' Postgres CDC connector, growing from a handful of users to 400+ replicating over 200 TB/month. The article is an honest field report on what it actually takes to make CDC reliable at scale — hundreds of edge cases, thousands of small capabilities, and the surprising amount of iteration needed to make the system feel boring.

---

## Key Quotes

> "the amount of iteration required to make the system feel *boring*"

This is the article's thesis in one sentence. CDC tools are easy to demo and brutally hard to productionize. "Boring" — meaning reliable, invisible, fast — is the highest praise an infrastructure engineer can give, and it takes vastly more work than anyone expects.

> "understanding the true complexity of data-movement / ETL systems" — that enterprise-grade CDC reliability depends on "hundreds, sometimes thousands, of smaller capabilities and edge cases working together"

Honest about what makes CDC hard. It's not the core algorithm (reading WAL is well-understood). It's the long tail: schema changes mid-replication, replication slot backpressure, long-running transactions at 2 AM, nullability propagation bugs. The difference between a prototype and a product is which edge cases you've been paged about.

> Partition generation went from "7+ hours to under a second"

The Cyera contribution (Alon Zeltser) replaced heavyweight COUNT/window-function queries with block-based partitioning on the CTID column. A customer hitting real scale found the bottleneck and fixed it — the kind of community dynamic that usually dies after acquisition but survived here.

> "on-call load dropped by orders of magnitude"

The team went from alerting on every individual error (YC "do things that don't scale") to bucketed, user-facing alerts across 10+ categories. A necessary pivot past ~100 customers. The note that they "remain in close contact with 10–20 customers at any time" is honest about the human cost of infrastructure reliability.

---

## Key Themes

#tool #concept #pattern #person

- **#tool** — [[ClickHouse]], PeerDB, ClickPipes, Postgres logical replication, Debezium
- **#concept** — **CDC (Change Data Capture):** deceptively deep. "Just read the WAL" hides long-running transactions, schema change propagation, replication slot management, and thousands of edge cases. **ReplacingMergeTree:** the #1 friction point for Postgres-on-ClickHouse users. It's a deduplication engine, not a real UPDATE, and the impedance mismatch dominates migration cost.
- **#pattern** — **Modular OSS + managed service:** Keeping PeerDB free and open as a standalone component while building ClickPipes CDC as the managed layer on top. Sai calls this "critical for engineering velocity." **Community contribution at scale:** Cyera contributing a fundamental fix (CTID-based partitioning) shows the model working past acquisition.
- **#person** — Sai Srirampur (PeerDB founder, now ClickHouse), Alon Zeltser (Cyera, contributed the block-based partitioning fix)

---

## Critical Analysis

**The "keep it open" playbook actually worked here.** Most acquired OSS projects wither — the founder leaves, the community scatters, the repo goes stale. PeerDB didn't. Part of that is keeping the OSS repo distinct from the managed product, and part is that ClickHouse genuinely needed this capability. But the Cyera contribution is the real signal: a customer at scale found a 7-hour bottleneck and fixed it upstream. That's a healthy project.

**The data modeling gap is the real story.** ReplacingMergeTree as the #1 pain point reveals a fundamental tension: ClickHouse wasn't designed for mutable data. Postgres users expect UPDATE and DELETE to just work. Asking them to rebuild their mental model around deduplication-at-read-time is a hard sell. Sai's roadmap (lightweight UPDATE, Postgres-compatible layer, JOIN improvements) acknowledges this, but it's a multi-year project. Compare [[AliSQL]], which takes the opposite approach: add OLAP to MySQL rather than teach OLAP users to think differently.

**AI workloads are accelerating the timeline.** The claim that "time to grow beyond terabyte-scale has shrunk from multiple years to just a few months" is specific and alarming. If AI-native companies are hitting Postgres analytical limits in months instead of years, the CDC-to-analytics-DB pipeline stops being a nice-to-have and becomes a mandatory architectural component. This is a tailwind for ClickHouse, but also for every other analytical database.

**"Make it boring" is the right framing, but the article underplays how far they still are.** 50+ preflight checks, bucketized alerts, Terraform "planned for next quarter" — this is a system in active professionalization, not a finished product. The honesty about footguns (schema change gaps, nullability bugs, uninterruptible operations) is refreshing, but someone evaluating this for production should read those sections carefully. CDC tools live or die by their edge cases, not their happy path.

**The long-term vision is the interesting part.** "Unify these two amazing databases as components of a single stack rather than separate databases" — this is the convergent database thesis again (see [[AliSQL]]). But ClickHouse is approaching it from the other direction: instead of adding columnar storage to a row-oriented engine, they're adding row-oriented semantics (UPDATE, unique indexes, Postgres compatibility) to a columnar engine. Both paths are hard. The question is which architecture has the better foundation for the convergence.

**Inside a database, CDC is a solved problem because there's a log.** Between companies, as [[The Valley of Webhooks]] argues, we're still rebuilding it from doorbells — webhooks and polled list APIs, one bespoke connector at a time, sold as product by Fivetran and Airbyte. The PeerDB/ClickHouse story is what reliable replication looks like when you have WAL access. The webhook ecosystem is what it looks like when you don't.

---

*Sources: [[summary/postgres-cdc-clickhouse-year-in-review-2025]]*
*Last updated: 2026-05-14*
