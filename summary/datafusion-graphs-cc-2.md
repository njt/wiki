---
url: https://semyonsinchenko.github.io/ssinchenko/post/datafusion-graphs-cc-2/
title: "Algorithms on billion-scale graph using 10GB RAM: I love DataFusion!"
author: Sem Sinchenko
date_fetched: 2026-08-01
date_published: 2026-07-05
topics:
  - databases-and-data
---

Sem Sinchenko demonstrates running graph algorithms — PageRank and weakly
connected components — on billion-edge datasets from a laptop, using Apache
DataFusion as the compute engine. The key insight is that DataFusion's
spill-to-disk, sort-merge joins, and aggregation machinery do the heavy lifting
that previously required a Spark cluster.

PageRank on the graph500-26 dataset (32M nodes, 1B edges) completes 15
iterations in ~30 minutes under a 5 GB memory cap. WCC on the twitter_mpi
dataset (53M nodes, 2B edges) finishes in 10 minutes with a 10 GB cap, after
symmetrizing and contracting edges across 22 forward iterations. Both results
were verified against Graphalytics ground truth.

The implementation follows the Pregel bulk-synchronous parallel model,
expressed through DataFusion joins and aggregations — similar in spirit to
Spark's GraphFrames. Edges and vertices are represented as DataFrames in a
GraphFrame struct. The code lives at github.com/SemyonSinchenko/graphframes-rs
and was written by hand (no LLM-generated core logic), as the author is
learning Rust and DataFusion through the project.

Known rough edges: FairSpillPool deadlocks under extreme memory pressure, and
no way yet to make sort-merge joins reuse pre-sorted on-disk data across
iterations. The author, who previously wrote skeptically about DataFusion for
graphs, now considers it a viable laptop-scale alternative to Spark +
GraphFrames.
