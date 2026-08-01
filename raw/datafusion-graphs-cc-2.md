---
url: https://semyonsinchenko.github.io/ssinchenko/post/datafusion-graphs-cc-2/
title: "Algorithms on billion-scale graph using 10GB RAM: I love DataFusion!"
author: Sem Sinchenko
date_fetched: 2026-08-01
date_published: 2026-07-05
tags: Rust, DataFusion, Graphs, Connected-Components, PageRank, Pregel
---

# Algorithms on billion-scale graph using 10GB RAM: I love DataFusion!

**Author:** Sem Sinchenko
**Published:** July 5, 2026 · 7 min read
**Tags:** Rust, DataFusion, Graphs, Connected-Components

## TLDR / Core Thesis

The author implemented graph map-reduce algorithms using Apache DataFusion, offloading data to disk and relying on bulk scans rather than random access. DataFusion handles spillover, sort-merge joins, aggregations, planning, and execution, keeping the custom code lightweight. Testing was done in strict mode via `systemd-run` with hard memory limits.

Key claim: "I can compute PageRank on a directed graph with one billion edges using 5 GB of memory" and identify weakly connected components in a two-billion-edge graph using 10 GB — things neither NetworkX nor Igraph can do. The author says they previously thought Spark + GraphFrames were needed for billion-scale graph analytics, but now believe "all you need is a laptop."

---

## Setup

### PageRank Task
- Dataset: `graph500-26` from Graphalytics
- 32,804,978 nodes, 1,051,922,853 edges
- Memory limit: 5 GB; DataFusion pool: 4 GB

Implementation is classical Pregel (bulk-synchronous parallel / map-reduce) expressed via joins and aggregates, similar to Spark's GraphFrames core.

### Weakly Connected Components (WCC) Task
- Dataset: `twitter_mpi`
- 52,579,682 nodes, 1,963,263,821 edges (directed)
- Memory limit: 10 GB; DataFusion pool: 8 GB

WCC implementation is based on "In-database connected component analysis," Bögeholz et al., arXiv 1802.09478.

---

## Results

### PageRank
- Used sort-merge join (SMJ) to prove scalability; hash join (HJ) is faster since vertices are small (32M) and state is just `rank` (f64), `out-degree` (i64), and a participation flag (bool).
- ~30 minutes for 15 full iterations; the author notes the setup is about memory, not speed.
- Results verified against ground truth: 100% match at 0.0001 tolerance.

Author's noted optimization ideas: bucket edges by range so SMJ doesn't re-sort the largest join side each iteration; question whether Parquet is optimal; explore fusing join+agg stages in DataFusion.

### WCC

The hardest part. Edges are 30 GB in CSV. Symmetrization (union of src/dst + distinct) peaks at ~4 billion edges using only an 8 GB pool. After the first few iterations, contraction drops edges dramatically; algorithm finishes in 10 minutes.

The author ran it under a hard 10 GB memory cap. Log excerpt shows preparation yielding 3,228,212,374 edges, then forward iterations shrinking from 840,238,268 edges (iteration 1) down to 0 (iteration 22), followed by back-propagation steps t=21 down to t=1. Final log line: "connected components written to ... after 22 forward iterations."

Correctness check against Graphalytics ground truth — both ground truth and results show the largest component has 52,515,193 vertices, and the next four components have 67, 44, 33, and 30 vertices respectively. The author verified via DuckDB queries comparing `twitter_mpi-WCC` against their results.

---

## The Code

Repository: https://github.com/SemyonSinchenko/graphframes-rs

The author notes: "I wrote most of the code by myself, not 'Claude do it, make no mistakes'", and states no LLM was used in core parts; they're learning Rust and DataFusion on this project.

Graph representation (similar to Spark's GraphFrames):

```rust
#[derive(Debug, Clone)]
pub struct GraphFrame {
    pub(crate) vertices: DataFrame,
    pub(crate) edges: DataFrame,
}
```

Core files:
1. Pregel main loop: `src/algorithm/pregel.rs`
2. WCC implementation: `src/algorithm/connectivity/connected_components.rs`
3. PageRank example: `src/algorithm/centrality/pagerank.rs`

Reference material cited: Stanford CME 323 Lecture 8 (Pregel paradigm).

---

## Key Points

- DataFusion's spill-to-disk, sort-merge joins, and aggregations enable billion-scale graph processing on a laptop-class machine.
- Known issues encountered: deadlocks from `FairSpillPool` in extreme scenarios; no way found yet to make SMJ use pre-sorted on-disk data.
- WCC command example used `systemd-run` with `-p MemoryMax=10G -p MemorySwapMax=0` and two CPUs.
- The author previously wrote a skeptical post about DataFusion for graph analytics and has now "completely changed" that opinion.
