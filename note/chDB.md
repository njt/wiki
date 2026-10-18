# chDB

chDB is ClickHouse as an embedded library: the full in-process SQL OLAP engine with no server to install, zero dependencies, and bindings for Python, Go, Rust, NodeJS, Bun, and C/C++. Its new DataStore API wraps the engine in a pandas-compatible surface — 630+ methods, lazy evaluation compiled to optimized SQL, SQL pushdown to the data source — pitching a "one-line migration" from pandas with up to 20× speedup at 10M rows.

---

The page is a documentation index, not an essay, so the substance is in the claims:

- **In-process OLAP**: ClickHouse's engine, embeddable, "no need to install ClickHouse server". This places chDB squarely in the embedded-analytical-engine niche DuckDB created.
- **Formats**: input/output for Parquet, CSV, JSON, Arrow, ORC "and 70+ more".
- **DataStore**: "pandas-compatible API that combines familiar pandas syntax with ClickHouse performance" — lazy evaluation, SQL pushdown, 209 compatible DataFrame methods plus 185+ accessor methods, universal sources (files, S3, databases, data lakes).
- **Benchmarks referenced but not shown here**: ClickBench of embedded engines, a DataFrame benchmark, and "up to 20x faster than pandas on common operations" (10M rows).

**Critical take.** This is a competitive announcement aimed directly at two audiences at once: DuckDB (same embedded-OLAP pitch, columnar, format-plural, zero-dependency) and pandas itself (DataStore is an adoption funnel for the world's default DataFrame library). Both pitches are unproven on this page: the "20× faster" number is unreproducible here, and pandas compatibility is always partial — 209 of ~250 DataFrame methods means the long tail (multi-index gymnastics, odd dtypes) is where the one-line migration will snag. The "Key Differences" doc link is doing quiet, important work. Still, the strategic move is real: ClickHouse taking its columnar engine to where the data scientists already are, in-process, rather than asking them to run a server — the same "bring the engine to the data" logic that made DuckDB the default laptop analytics tool. Whether chDB's ecosystem maturity matches DuckDB's is the open question.

**Themes**: #tool #pattern (embedded OLAP, pandas-compatibility-as-funnel) #concept (SQL pushdown, lazy evaluation)

**Related pages:**

- [[Wes McKinney on Pandas, Arrow, and Data Infrastructure]] — McKinney's "full-circle return from distributed back to single-machine columnar engines" is exactly the trend chDB's DataStore rides: it tries to capture pandas users by making the columnar engine invisible under the familiar API.
- [[Apache DataFusion]] — the other embeddable-engine story: DataFusion is the neutral, Rust/Arrow substrate engines are built on; chDB is the vendor-proprietary-engine counterpart, embedding a battle-tested C++ engine rather than building from the open stack.
- [[DuckDB ADBC Extension]] — DuckDB's move was standardizing connectivity (ADBC) across the columnar ecosystem; chDB's move is standardizing the *API surface* (pandas). Two different wedges into the same "engine comes to you" future.
- [[What Is Columnar Storage (MotherDuck)]] — the mechanical explanation of why a columnar engine like ClickHouse inside chDB beats row-wise pandas on the aggregations DataStore pushes down.

---
*Sources: [[raw/index]], [[summary/index]]*
*Last updated: 2026-10-18*
