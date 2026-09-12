# DataFusion for Billion-Scale Graph Algorithms

Sem Sinchenko's field report on using Apache DataFusion — not Spark, not a GPU cluster — to run PageRank on a billion-edge graph in 5GB RAM and weakly connected components on a two-billion-edge graph in 10GB RAM, on a laptop. The thesis is simple and radical: the query engine handles spill-to-disk, sort-merge joins, and aggregation; you write a Pregel loop in ~100 lines of Rust. The result is an existence proof that single-machine graph analytics at billion-scale is not just possible but practical.

---

## Key Quotes

> "A few months ago I wrote a skeptical post about DataFusion for graph analytics… Now I can say: I have COMPLETELY changed that opinion!"

The author's journey is part of the story. The skepticism-to-conversion arc mirrors what happened with DuckDB for OLAP — people didn't believe a single process could beat a cluster until someone ran the numbers.

> "All you need is a laptop."

This is the COST paper's thesis applied to graph analytics. For a specific class of problems — PageRank, connected components, label propagation — the coordination overhead of distributed systems isn't buying you anything. A single machine with a smart query optimizer and spill-to-disk can process two billion edges in ten minutes.

> "I wrote most of the code by myself, not 'Claude do it, make no mistakes'"

A deliberate disclosure. The author is learning Rust and DataFusion by building, not by prompting. The code is a learning artifact as much as a tool. This matters because it means the design decisions are legible — you can trace why the Pregel loop looks the way it does, rather than inheriting an opaque agent-generated architecture.

## Key Themes

**#concept — Map-reduce as the graph algorithm primitive.** Pregel (bulk-synchronous parallel) is naturally expressed as join → aggregate → repeat. DataFusion's query planner and execution engine handle the hard parts: spilling, sorting, memory management. The custom code is thin — a few hundred lines of Rust that express the algorithm, not the infrastructure.

**#tool — DataFusion as a graph engine.** DataFusion was designed for SQL analytics on Arrow data, not graph processing. But its primitives (joins, aggregations, spill-to-disk, sort-merge) happen to be exactly what iterative graph algorithms need. This is the Apache Arrow ecosystem's quiet victory: a general-purpose columnar query engine turns out to be competitive with specialized graph frameworks.

**#pattern — Bulk scans beat random access at scale.** Graph algorithms on billion-scale graphs can't do random vertex lookups — the working set doesn't fit in memory. The solution is to express everything as bulk scans with joins, which is what relational engines have been optimizing for decades. The insight isn't new (it's how Spark GraphFrames works), but proving it works on a single laptop with 10GB RAM is.

**#pattern — systemd-run as a benchmarking harness.** Using `systemd-run --scope -p MemoryMax=10G -p MemorySwapMax=0` to enforce hard memory caps during testing is a clever, zero-dependency approach. No containers, no cgroups scripting — just Linux's init system doing resource control. A reminder that good benchmarking infrastructure is often simpler than we think.

## Critical Analysis

**The real contribution is the existence proof, not the code.** Nobody doubted that join-based graph algorithms *could* work at scale — Spark GraphFrames has done this for years. The surprise is that a single Rust binary with DataFusion can do it on consumer hardware. The author ran on *two CPUs* with 10GB RAM. This is the graph-analytics equivalent of "your distributed system is slower than a laptop."

**The "laptop" framing overstates the case slightly.** The graphs tested (graph500-26, twitter_mpi) are large but not the largest. A trillion-edge web graph would still need a cluster. But the author's point stands for the 99% use case: most organizations' graph data fits on a single beefy machine, and they're paying for distributed infrastructure they don't need.

**The LLM disclosure is a trust signal.** The author is explicit about not using AI to write the core code, which means the design reflects human reasoning about tradeoffs. In a landscape where AI-generated code is increasingly opaque about its provenance, this is worth noting. It also means the code is a better learning resource — you're reading decisions, not statistical averages.

**The FairSpillPool deadlock problem is the open wound.** DataFusion's memory management works until it doesn't. Under extreme pressure (4 billion row distinct), the spill pool can deadlock. The author doesn't have a fix, and neither does DataFusion. This is the kind of edge case that separates "works in benchmarks" from "works in production." For anyone considering this approach seriously, this is where you'll spend your debugging time.

**This is the Arrow ecosystem's compounding return.** DataFusion benefits from Arrow's columnar format, Parquet's compression, and years of query optimization research poured into a reusable Rust library. None of this was built for graph processing. It just turns out that bulk-synchronous graph algorithms look a lot like iterative SQL, and DataFusion is very good at iterative SQL.

## Connections

- [[Wes McKinney on Pandas, Arrow, and Data Infrastructure]] — Wes calls Arrow adoption a "decade-long" process, and DataFusion proving itself on graph workloads is exactly the kind of unexpected compounding return he describes.
- [[Your Distributed System Is Slower Than a Laptop]] — The COST paper's core argument applied to graph analytics: single-machine with smart engineering beats a cluster for more workloads than the industry admits.
- [[DuckDB ADBC Extension]] — DuckDB and DataFusion are sister projects in the Arrow ecosystem; DuckDB took the embedded OLAP path while DataFusion became the embeddable query engine. Both prove the same thesis: single-process columnar engines are the right answer for most data problems.
- [[The GUS Stack — Go, Unix, SQLite]] — The philosophical cousin: choose boring, well-optimized single-machine infrastructure over distributed complexity. DataFusion on a laptop is that argument for graph processing.
- [[Databases and Data]] — Hub page for data infrastructure.

---
*Sources: [[raw/datafusion-graphs-cc-2]]*
*Last updated: 2026-08-01*
