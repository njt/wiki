# Random Access Parquet (RAP)

Spotify's technique for turning the data lake into an online serving layer: an external index maps each key directly to its Parquet file, rows, and pages, so a point query collapses from a chain of dependent reads (footer → row-group metadata → key column → page indexes) into one O(1) index lookup plus a few parallel ranged reads. The same files that feed batch analytics serve interactive lookups — no copy into a KV store, no ETL, no second storage bill.

---

## The Core Insight

The article's sharpest move is reframing where the bottleneck lives. Object storage (GCS at 30–100ms, S3 Express One Zone and GCS Rapid Storage at single-digit milliseconds) is no longer the slow part — the query engine is. Trino and BigQuery add seconds of scheduling overhead even for a one-row lookup because they're built for analytical throughput, not point queries.

> "The bottleneck is increasingly not the storage layer itself, but the query engines on top."

RAP's answer is to stop scanning and start looking up. Where Parquet's built-in PageIndex and Bloom filters are *probabilistic* and merely narrow a scan, the external index is *definitive* — it names the exact files and rows, eliminating the scan entirely.

> "Collapsing that chain — replacing dependent reads with a single precomputed lookup — saves both latency and bandwidth regardless of the storage tier."

The structural insight generalizes: the absolute cost of each read link differs (tens of milliseconds on cloud storage, microseconds on SSD, nanoseconds in memory), but the *shape* is identical — each step waits on the last. Killing the dependency chain is the win at every tier. This is the same "collapsed dependency" logic that powers the [[The Log — Unifying Abstraction for Real-Time Data]] worldview, applied to the read path instead of the write path.

## The Optimization Catalog

The heart of the piece is a taxonomy of write-time Parquet tricks, grouped into three families:

- **Concentrate a key's data** — sort by key, co-group (one row per key with repeated/nested columns), coarser partitioning. Fewer files and pages per key.
- **Reduce bytes per read** — one-page-per-key (flush pages at key boundaries), ZSTD frame resets within pages (each key's rows are an independently addressable ZSTD frame), storage alignment via skippable frames.
- **Reduce read operations** — blobs/variants (one column per read), interleaving columns (pivot to row-major for a column group), covering indexes that hoist small values straight into the index so the storage read becomes zero.

> "Reducing the read count is the biggest single win."

This is honest engineering, not marketing gloss: the summary table lists a tradeoff for every technique. One-page-per-key grows the PageIndex. ZSTD frame resets rule out delta/run-length encoding. Interleaving makes single-column scans read dead space. Blobs kill per-field column pruning for batch. RAP doesn't make point queries free — it moves the cost into the write path and the index, and it *changes which tradeoffs matter*. Once the index tells you the exact page before you touch the file, fine-grained in-file discovery stops paying for itself, and minimizing the final fetch does.

## One Dataset, Two Access Patterns

The economic claim is the real product. Today teams ration KV-store space because per-GB cost forces prioritization of what gets served online. RAP drops a point query's cost to "a cloud storage read," which makes historical data, long-tail entities, and low-traffic features all viable for interactive access.

> "The files that BigQuery scans for weekly aggregate reports are the same files that an AI agent reads for context retrieval. No copy, no ETL, no second storage bill."

> "The data lake is no longer batch-only. The same storage serves both analytical and interactive workloads — one dataset, two access patterns."

The AI-agent motivation is more than a garnish. An agent answering "what was I listening to last summer?" needs exactly this primitive: O(1) key → rows, then filter/aggregate/SQL locally to build LLM prompt context. RAP is a retrieval layer pointed at the lake rather than a KV or embedding store.

## Critical Analysis

**RAP is the read-path counterpart to LTAP, and they complete each other.** [[Lakebase and LTAP]] solves the write side — getting operational data *into* open columnar formats without a CDC pipeline. RAP solves the read side — getting it *out* fast for point queries. Read together they sketch a full "storage-layer unification" thesis: write once in Parquet, serve transactions, analytics, and now interactive lookups from the same governed copy. The article never says this, but it's the same move.

**"Store once, pay once" is true for the data, not for the index.** RAP builds a new artifact — gigabytes of index per terabyte, terabytes per petabyte — that must stay consistent with the lake as it grows. The article is candid that the index is appended as "fragments" per pipeline run, but the freshness question is left hanging. A "what did I listen to last summer" query tolerates a lagging index; a "what did I listen to five minutes ago" query does not. Index propagation latency is RAP's analog of LTAP's unstated materialization SLA.

**It's vendor-neutral in tone but it's a Spotify post.** RAP "operates on any existing Parquet files with no special preparation" — a strong portability claim. But the prepared-file optimizations (ZSTD frame resets, interleaving, one-page-per-key) require controlling the write pipeline, and the index itself is serving infrastructure someone has to build and operate. The unmodified-file path works immediately; the real latency and cost wins live behind pipeline work, which is where the effort actually is.

**Secondary indexes are hand-waved.** "Adding or removing a secondary index is a serving-layer decision — no pipeline changes, no data rewriting" is a genuine benefit, but the space-filling-curve aside (Z-ordering, Hilbert) being "complementary" does a lot of quiet work. Scattering a secondary-dimension lookup across many files and then "coalescing adjacent byte ranges" is a real systems problem, not a footnote.

**The biggest unstated dependency is the "cached file metadata."** The whole approach assumes page locations can be resolved from cached metadata without a round-trip. For a lake of thousands of daily files across thousands of columns, keeping that metadata hot is itself a distributed-cache problem that the article mentions only in passing.

## Connections

This is the interactive-lookup pole of the columnar story [[Wes McKinney on Pandas, Arrow, and Data Infrastructure]] tells: the last-mile problem of moving data between systems doesn't go away, it just moves. RAP's answer to the "serialization boundary" is to not cross it — point the query at the lake file itself. It also rhymes with [[Icebug Format]]'s trick of querying Parquet in-place from object storage, and with [[Databases and Data]]'s convergent-architectures observation — here the convergence is a single storage format serving two access patterns rather than a single engine doing two jobs. A close cousin is [[Drilldown Dashboards from a Single Parquet File]], which achieves the same store-once-serve-interactive trick with no external index at all: precomputed GROUPING SETS sorted into one Parquet cube, where the file's own row-group min/max statistics prune a 40MB cube down to a 260KB read per filter click — free pruning that only works for bounded aggregate questions, whereas RAP's paid-for index answers arbitrary point lookups.

---
*Sources: [[raw/indexing-the-data-lake-for-online-point-queries]], [[summary/indexing-the-data-lake-for-online-point-queries]]*
*Last updated: 2026-08-21*
