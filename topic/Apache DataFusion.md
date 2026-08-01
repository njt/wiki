# Apache DataFusion

Apache DataFusion is an extensible, embeddable SQL query engine written in Rust, built on Apache Arrow's columnar in-memory format. It is not a database — it is the engine you build a database *on top of*. Think of it as the query execution layer that DuckDB, ClickHouse, or a custom analytical system might embed, with customization points at nearly every level: data sources, query languages, functions, operators, and the optimizer itself.

---

## What It Is (and Isn't)

DataFusion sits at the infrastructure layer, not the end-user layer. End users interact with it indirectly through projects that embed it: the Python and Java bindings, the Comet Spark accelerator, and the Ballista distributed runtime. The core project is a Rust library (published on crates.io) plus a CLI binary. It is not a database you connect to; it is a library you compile into your own system.

This matters because it makes DataFusion a **platform play** for the Arrow ecosystem. Where DuckDB is "SQLite for analytics" — a self-contained embeddable database — DataFusion is the query engine toolkit for building DuckDB-like things, each customized to a specific workload.

> "an extensible query engine written in Rust that uses Apache Arrow as its in-memory format"

The first clause ("extensible") is doing the heavy lifting. DataFusion's architecture is customization-first: plug in your own data sources, your own SQL extensions, your own optimizer rules, your own table providers. The canonical comparison is to LLVM for compilers — not a compiler itself, but the reusable infrastructure that makes building compilers tractable.

## Architecture

DataFusion provides a complete query pipeline, each stage replaceable:

1. **SQL parser** (based on sqlparser-rs) → AST
2. **Query planner** → logical plan with rule-based and cost-based optimization
3. **Physical planner** → physical execution plan
4. **Execution engine** — columnar, streaming, multi-threaded, vectorized, with partitioned data sources

The execution engine processes Arrow columnar batches through a Volcano-style iterator model, with vectorized operations running across multiple threads. Data sources are partitioned, allowing the engine to parallelize scans across CPU cores without any coordination overhead.

## Key Themes

**#tool Embeddable over standalone.** DataFusion is not competing with databases — it is competing with the idea that every analytical system should build its own query engine from scratch. The embeddable strategy is the same one that made SQLite ubiquitous: be the component, not the product.

**#pattern Platform over product.** The Arrow ecosystem ([[Wes McKinney on Pandas, Arrow, and Data Infrastructure]]) now has three layers: Arrow itself (the in-memory format), Parquet (the on-disk format), and DataFusion (the query engine). Together they form a complete, open-source columnar analytics stack with no proprietary dependencies. Every new engine that adopts this stack — DuckDB, Polars, Daft, InfluxDB 3.0 — reinforces the standard.

**#tool Extensibility as architecture.** Where most query engines let you add UDFs, DataFusion lets you replace the optimizer, add table providers, define custom operators, and extend the SQL dialect. This is a Rust trait system applied to query engine design: define the interface, let users provide the implementation. The trade-off is complexity — the Library User Guide spans versions 46 through 55, each with breaking changes to the extension API.

**#concept The Rust data infrastructure convergence.** DataFusion, Polars, Daft, and InfluxDB 3.0 are all Rust query engines built on Arrow. Rust's combination of performance, safety, and expressiveness makes it the consensus choice for new analytical infrastructure. The [[The GUS Stack — Go, Unix, SQLite]] thesis — use boring, training-data-dense technologies — applies here: Rust is becoming boring in exactly the right way for data infrastructure.

## The Subproject Ecosystem

DataFusion's subprojects map the adoption spectrum from "just use it" to "build with it":

- **DataFusion Python** — The bridge to the Python data science ecosystem. If you're a Python user who wants DataFrame and SQL queries without running a database server, this is the entry point.
- **DataFusion Comet** — A Spark accelerator. Instead of rewriting your Spark jobs, Comet speeds up the existing ones by replacing Spark's JVM-based execution with DataFusion's Rust engine. Pragmatic: don't ask users to migrate, just make their current thing faster.
- **DataFusion Ballista** — Distributed DataFusion. Splits query execution across multiple nodes. This is where DataFusion graduates from single-machine to cluster-scale, but it's the least mature of the subprojects. Distributed query execution is a harder problem than most people appreciate, and the gap between "works on a demo" and "works when a node fails mid-query" is measured in years.

## Critical Analysis

**The embeddable bet is correct but quiet.** Embeddable engines (SQLite, DuckDB, DataFusion) are eating the world from the bottom up because they eliminate operational complexity. You don't need a DBA to run a query engine compiled into your binary. The trade-off is mindshare: embeddable projects don't have dashboards, don't have marketing pages comparing TPC-H benchmarks, and don't show up in CIO evaluation matrices. DataFusion's obscurity relative to its importance is a function of being infrastructure, not a product.

**The extension API churn is a real cost.** The Library User Guide documents upgrades from version 46 through 55 — that's nine breaking-change releases. Each one requires downstream projects (InfluxDB, GreptimeDB, risingwave, etc.) to update their integration code. This is the cost of being a platform: your API stability *is* your product stability. DataFusion's rapid evolution is a sign of health (the project is actively improving) and a source of friction (the ground keeps moving). The Arrow specification took a decade to stabilize; DataFusion is on the same trajectory.

**Comet is the smartest thing in the portfolio.** Rather than competing with Spark — an ecosystem with 50,000+ companies invested — Comet makes Spark faster from the inside. This is the same strategy that made Arrow successful: don't ask people to switch, make their current stack better. If Comet succeeds, DataFusion gets adoption at Spark-scale without anyone having to choose DataFusion. It's the most elegant adoption strategy in the Arrow ecosystem.

**Ballista is the hardest problem.** Distributed SQL execution is the graveyard of ambitious query engine projects. The gap between single-node DataFusion (which works great) and multi-node Ballista (which has to handle partial failures, stragglers, network partitions, and clock skew) is not a gap — it's a different category of problem. [[Your Distributed System Is Slower Than a Laptop]] applies here: for most workloads, a single beefy machine running DataFusion will beat a cluster of smaller machines running Ballista, and with none of the coordination overhead.

**The Apache Foundation governance is a mixed blessing.** ASF provides legal shelter, trademark protection, and community norms. It also adds process overhead, consensus requirements, and the slowest possible decision-making. For a fast-moving Rust project, the tension between "move fast" and "Apache process" is real. The project's health suggests they've found a workable balance so far.

**Missing: a clear "build your own database" guide.** DataFusion's documentation is excellent at the API level but assumes you already know what kind of database you want to build. A "DataFusion for Database Builders" guide — walking through the decisions you face when embedding the engine (storage format, indexing strategy, catalog design, transaction model) — would dramatically lower the adoption bar. As it stands, you need to reverse-engineer the decisions made by InfluxDB 3.0 and GreptimeDB to understand the design space.

---

## Where It Fits

DataFusion completes the open-source columnar analytics stack. The layers:

| Layer | Technology | Role |
|-------|-----------|------|
| In-memory format | Apache Arrow | Zero-copy columnar data exchange |
| On-disk format | Apache Parquet | Compressed columnar storage |
| Query engine | Apache DataFusion | SQL + DataFrame execution |
| Embeddable DB | DuckDB | Full database with storage, transactions |
| Distributed DB | Ballista | Multi-node DataFusion |

Every layer is open-source, every layer uses Arrow as the wire format, and every layer is increasingly built in Rust. This is not a stack anyone designed top-down — it emerged from convergent evolution toward the same technical truths.

See also: [[Databases and Data]], [[Wes McKinney on Pandas, Arrow, and Data Infrastructure]], [[DuckDB ADBC Extension]], [[Streambed]], [[Your Distributed System Is Slower Than a Laptop]].

---
*Sources: [[raw/apache-datafusion]]*
*Last updated: 2026-08-01*
