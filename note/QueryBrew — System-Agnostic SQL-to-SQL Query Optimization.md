# QueryBrew — System-Agnostic SQL-to-SQL Query Optimization

VLDB 2026 demo paper from the TU München group behind Umbra (Schmidt, Reif, Birler, Neumann). QueryBrew is an "optimizer as a service" that decouples query optimization from the database engine by rewriting SQL to SQL: arbitrary SQL goes in, Umbra's optimizer reworks it on relational algebra (general unnesting, simplification, adaptive join reordering), and the optimized plan is distilled back into "operator-oriented SQL" — one CTE per plan operator. PostgreSQL, ClickHouse, DuckDB, and SQL Server then execute the rewrite and inherit optimizations they never shipped, with speedups of 19.9–21.8× on a correlated TPC-DS query and >100× on some JOB joins. The quiet radicalism is using SQL itself as the plan interchange format instead of an IR like Substrait, which the authors argue is precisely why this decoupling can succeed where IRs stalled.

---

## The move: SQL instead of an IR

The obvious way to share an optimizer across engines is a common plan representation. Substrait tried; it "has not seen wide adoption due to the inherent difficulty of unifying the semantics of existing systems with their various idiosyncrasies," so modular efforts like CompoDB support only a few engines (DuckDB, DataFusion, Acero). Microsoft's QOaaS route passes Substrait-to-Substrait plans from Fabric's Unified Query Optimizer to Spark. QueryBrew skips the IR entirely: SQL is already the interface every engine parses, so make it the interchange format in both directions. The target system's optimizer still runs on top — picking physical operators, indexes, storage-specific tricks — but on a structurally simpler, already-decorrelated query.

The paper's sharpest evidence that optimizer coupling is the real problem, not syntax:

> "Even the many database solutions at Google, such as BigQuery, Spanner, F1, BigTable, Dremel, and Procella, share the SQL frontend GoogleSQL but do not share a single optimizer."

One company, total control of its stack, a shared frontend — and still no shared optimizer. If Google can't extract its own optimizer, an IR won't either; the language everyone already agrees on is SQL.

## One CTE per operator

The back-translation encodes every operator of Umbra's optimized plan (a DAG, not a tree) as a named CTE that references its inputs' CTEs: `scan_1`, `groupby_2`, … The encoding preserves execution order, including the build/probe sides of joins, and join order is pinned with system-specific settings and hints where possible so the target optimizer can't undo Umbra's plan. Non-standard Umbra operators (group-joins, mark-joins, magic sets) need special care. The example walkthrough shows a real optimization in the output: `count(distinct i_color)` split into two group-bys — one to compute distinct category/color combinations, one to count them.

> "Hence, through SQL, we achieve a full decoupling of the optimizer from the query engine and storage: A perfect match for today's open table formats and multi-engine landscape."

The statistics detail is what makes "full" defensible. An external service can't collect statistics at insert time, so QueryBrew derives them in SQL — HyperLogLog sketches for distinct counts, AMS sketches for join selectivity, samples for filter selectivity, built from native hash functions and aggregations. No hooks into the engine, no hooks into storage. (The paper never prices this: sketching your data from outside, per query, is not free.)

## Results

On a correlated TPC-DS query over an 18,000-row item table — human-readable, quadratic if executed naively — Umbra's general unnesting pays off everywhere:

| System | Original | Optimized | Speedup |
|---|---|---|---|
| PostgreSQL | 40.8 s | 2.1 s | 19.9× |
| ClickHouse | 4.4 s | 0.2 s | 21.8× |
| SQL Server | *(withheld — DeWitt clause)* | *(withheld)* | 2.84× |
| DuckDB | 99 ms | 12 ms | 8.07× |
| Umbra | 18 ms | 18 ms | 1.00× |

> "To our surprise, ClickHouse (version 25.11) returns the wrong result for the original query; however, running the optimized version produces the correct result. The simplified CTE representation does not trigger the incorrect code path in ClickHouse."

That's the buried headline. The same query in two textual shapes, run across four systems, is a differential-testing oracle — and it found a wrong-result bug in a major analytical engine. SQL-to-SQL rewriting isn't just a performance tool; it's a correctness surface.

The authors are honest about the flip side: the rigid CTE structure can hide optimizations from the target optimizer, causing "only minor slowdowns" (always <5× in their experiments), and users can always run the original query instead. It's a best-effort service, not a contract.

## Why optimizers lag

> "Changes to the optimizer can yield immense benefits, but they are also likely to break some customers' workloads in unexpected ways (e.g., by removing one of two mistakes that cancelled each other out). Thus, big players in the industry approach developments in the optimizer with a risk-averse, calculated approach. Unfortunately, this means that optimizers often lag behind the latest innovations."

The "two mistakes that cancelled each other out" line is the best one-sentence account of optimizer conservatism I've read. QueryBrew reframes the politics: instead of persuading vendors to risk their customers' workloads on a better optimizer, externalize optimization and let anyone pull state-of-the-art plans on demand — the vendor's untouched optimizer just executes a plan it didn't have to find.

## The LLM footnote

Two asides make this relevant beyond the database crowd:

> "While the query (on the left) is easy to write and understand for humans or LLMs, its naive execution leads to quadratic runtime due to the correlated subquery."

> "Both LLM-based [8] and human-centered [3] rewriting approaches have been proposed; however, these surface-level techniques operate on the query text or the abstract syntax tree. Since many sophisticated optimizations operate on relational algebra, the capabilities of such text-based techniques are inherently limited."

LLMs write exactly this naive, correlated shape — readable and quadratic — because the text is easy. The fix is not a better prompt or a bigger model: it's a classical optimizer standing between the draft and the engine. And symmetrically, LLM-based "query rewriting" hits a ceiling by construction: it can only see text, while unnesting and join enumeration live in the algebra. Draft with a model, optimize with algebra.

## Analysis

The elegance is in the hijack: rather than asking every engine to adopt a new IR, QueryBrew exploits the one interface they already share, and in doing so it takes exactly the high-value logical optimizations (unnesting, join order) while leaving the genuinely engine-specific parts (physical operators, indexes, storage layout) to the native optimizer. Clean division of labor.

The limitations are the demo-paper ones. The evaluation is a handful of queries at TPC-DS scale factor 1; the >100× claims are from selected JOB queries; per-query SQL-computed statistics, hint syntax per dialect, and the maintenance of four dialect backends all go unpriced. The DeWitt-clause footnote on SQL Server is a charming reminder of benchmark politics, but it also means you can't fully audit one column of the headline table.

The real bet is architectural: if the world converges on open table formats read by many engines, per-engine optimizers are the remaining differentiator — and Microsoft's internal QOaaS plus this paper are both evidence that optimizer-as-a-service is the next unbundling. QueryBrew's contribution is showing the interchange format doesn't need to be invented. It's SQL.

Tags: #tool #project #databases #query-optimization #sql #pattern

## Related

- **[[Apache DataFusion]]** — QueryBrew complicates the modular-engine story: DataFusion is one of the few engines the Substrait route (per the paper, via CompoDB) could support, which strengthens the case that embeddable engines are tractable — and simultaneously shows why the IR path stalled, since QueryBrew succeeded by routing around the IR entirely and speaking SQL directly.
- **[[Lakebase and LTAP]]** — Xin's one-copy-many-engines world is the landscape QueryBrew names as its "perfect match": when every engine reads the same open-format data, the optimizer becomes the remaining per-engine differentiator, and this paper argues it doesn't even have to live inside the engine.
- **[[ClickBench Playground]]** — the playground races ~100 databases behind one interface; QueryBrew is effectively a four-system differential bench with query plans attached, and its ClickHouse 25.11 wrong-result find is a concrete specimen of what multi-system comparison surfaces — and a caution for trusting any single vendor's numbers.
- **[[Text-to-SQL in the Real World]]** — this paper nuances the text-to-SQL failure story: LLM-drafted SQL isn't just semantically unreliable, it's *plan*-pathological (readable correlated queries that compile to quadratic execution), and the demonstrated remedy is a classical optimizer downstream of drafting — not, per its own related work, LLM-based rewriting, which by operating on text rather than algebra is "inherently limited."

---
*Sources: [[raw/p4494-schmidt-pdf]], [[summary/p4494-schmidt-pdf]]*
*Last updated: 2026-09-13*
