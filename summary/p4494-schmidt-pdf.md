---
url: https://www.vldb.org/pvldb/vol19/p4494-schmidt.pdf
title: "QueryBrew: System-Agnostic SQL-to-SQL Query Optimization"
author: Tobias Schmidt, Maximilian Reif, Altan Birler, Thomas Neumann
date_fetched: 2026-09-13
date_published: 2026
site: PVLDB Vol. 19, No. 12, pp. 4494–4497 (VLDB demonstration track)
topics:
  - databases-and-data
---

A four-page VLDB 2026 demonstration paper from the TU München group behind Umbra (Schmidt, Reif, Birler, Neumann). QueryBrew is an "optimizer as a service": it decouples query optimization from the database engine by rewriting SQL to SQL. An arbitrary input query is optimized by the state-of-the-art Umbra optimizer — general unnesting, operator simplification, adaptive join ordering, common-subtree elimination, over a DAG-structured plan — and the optimized plan is distilled back into "operator-oriented SQL": one CTE per plan operator, connected by CTE references. Any target system (PostgreSQL, ClickHouse, DuckDB, SQL Server) executes the result canonically and inherits optimizations it never implemented, while its own optimizer still applies system-specific physical optimization on top.

The motivation is that the query optimizer is the highest-leverage, highest-risk database component: changes "can yield immense benefits, but they are also likely to break some customers' workloads in unexpected ways," so industry optimizers lag research. Even Google's systems (BigQuery, Spanner, F1, BigTable, Dremel, Procella) share the GoogleSQL frontend but not a single optimizer. Prior decoupling attempts — the Substrait IR, CompoDB, Microsoft's Fabric "Query Optimizer as a Service" — stall on unifying engine semantics. QueryBrew's bet is that SQL itself, already every engine's lingua franca, is the interchange format that clears the adoption wall.

Because an external service cannot hook an engine's insert/update path, QueryBrew computes its cardinality statistics — HyperLogLog sketches for distinct counts, AMS sketches for join selectivity, samples for filters — in SQL, using native hash functions and aggregations. Hence "a full decoupling of the optimizer from the query engine and storage: A perfect match for today's open table formats and multi-engine landscape." On a correlated TPC-DS query over an 18,000-row table: PostgreSQL 40.8s → 2.1s (19.9×), ClickHouse 4.4s → 0.2s (21.8×), DuckDB 99 ms → 12 ms (8.07×), SQL Server 2.84× (absolute times withheld under the DeWitt clause), Umbra unchanged. Some JOB queries improve by more than 100×; observed slowdowns from the CTE representation stay under 5×, and users can fall back to the original query.

Side effects: SQL dialect translation (PostgreSQL dialect to the four targets) that, unlike SQLGlot, handles arbitrarily complex queries; a demo UI with unified/per-system query editors, result comparison, and plan views, preloading TPC-H, TPC-DS, SSB, and JOB; and a correctness surprise — ClickHouse 25.11 returns a wrong result for the original correlated query but the correct result for the optimized CTE form. LLM-based (GenRewrite) and human-centered (QueryBooster) rewriters are dismissed as surface-level text/AST manipulation: real optimization operates on relational algebra.
