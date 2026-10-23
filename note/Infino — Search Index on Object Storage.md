# Infino — Search Index on Object Storage

Infino's write-up of what breaks when a Lucene-style full-text search index is moved from local disk to S3: every random read becomes a billed network round-trip, so the whole storage layer is redesigned to minimise GETs — while the search core (postings, BM25, block-max pruning) stays pure Lucene thinking.

---

Infino is building a retrieval engine that answers agent questions at scale — SQL, vector search, and BM25 over data that already sits in the object store. The post is part 1 of a series on how the engine is built, and its organising insight is a cost-model swap rather than an algorithmic novelty: on SSD, a random read is sub-millisecond; on S3, it's tens of milliseconds and billed per request. "The scarcest resource is not CPU, and not bytes, but the number of round trips you make to the object store."

The design falls out of that constraint with unusual cleanliness:

- **One object per index unit.** Lucene segments are many files (`.tim`, `.tip`, `.doc`, `.pos`, …); Infino's *superfile* is a single plain Parquet file with full-text and vector indexes spliced in as byte regions before the footer. Any Parquet reader (DuckDB, PyArrow) can open it.
- **A manifest that inlines metadata.** Each supertable's manifest stores every superfile's Parquet footer and index headers, so opening a superfile costs *zero* GETs — cold cost per superfile drops from two round trips to one. Column stats in the manifest also enable superfile skipping on filters like `created_at > '2026-10-01'`.
- **A term index that answers "which superfiles?"** Per term: containing superfiles, document counts, BM25 score bounds, and postings byte offsets. Top-k queries walk superfiles in score-bound order and prune with WAND-style logic; single-term counts are answered without opening any file — in one test, a count over 141 superfiles went from opening 141 to opening none.
- **Ranged, coalesced, cached reads.** Lazy byte-range fetches; postings within 64 KB coalesced into one GET; background full-file download to a disk cache (10 GB default, LRU-evicted), after which queries hit memory-mapped pages.

## Key quotes

> The scarcest resource is not CPU, and not bytes, but the number of round trips you make to the object store.

The whole post is this sentence applied repeatedly. It's a good model for any storage-layer port: identify the one resource whose price changed by orders of magnitude, then re-derive the architecture.

> However, the core of the search engine did not change. Postings lists, BM25 scoring, block-max pruning and the query algorithms are the same ideas Lucene uses... What changed is the storage layer, because that is where the cost model changed.

Honest and refreshing — most "we rebuilt X for the cloud era" posts oversell the novelty. Infino explicitly credits Lucene and frames its contribution as cost-model adaptation.

> A single-term count on a supertable with 141 superfiles went from opening 141 superfiles to opening none.

The term index's document counts turn a metadata problem into a free answer. The elegant part is that the same counts double as BM25's corpus-wide statistics — one data structure serving pruning, counting, and scoring.

## Themes

#concept (cost model as design driver) #tool (Infino) #pattern (metadata inlining to eliminate round trips)

## Analysis

The strongest thing here is the discipline of the framing. Rather than "object storage is slow, here's our cache," the post derives each decision from one measurable cost, and even footnotes the trade-offs it accepts (a superfile may be read twice — once by ranged GETs, once by the background full download; counts are approximate until deletes are optimised away). That candour is rare in vendor engineering blogs.

Worth noting: Infino is, self-admittedly, a product pitch — the benchmark comparison cited is hosted on Infino's own site, so the "among the fastest available" claim should be read as marketing-adjacent. But the architectural reasoning stands on its own regardless of whose engine wins the benchmark.

There is also a quiet second story here: Infino exists because *agents* need retrieval over data that already lives in the object store. The engine's design is a downstream consequence of the agent-era deployment reality — data gravity, multi-modal queries (SQL + vectors + BM25 in one engine), and the economics of per-request billing at agent query volumes.

## Relation to the wiki

- [[Random Access Parquet (RAP)]] — same underlying bet that Parquet can carry more than columnar row data; Infino splices index byte regions into the footer, RAP pushes random access further. The two strengthen each other's case that Parquet is becoming a general-purpose container format, not just analytics.
- [[Three Kinds of Agentic Search]] — Turnbull's retrieval-centric strategy ("build good search") is exactly what Infino is building; this post shows the retrieval-centric path at the storage-engine layer rather than the harness layer.
- [[AI Team Mistakes]] — Turnbull's "AI teams are search teams" thesis gets concrete engineering substance here: the search team's hard problem turned out to be round trips, not embeddings.
- [[Fluree DB]] — another engine integrating BM25 and vector indexes natively alongside its primary storage format; useful comparison point for how different systems absorb index structures into their file formats.

---
*Sources: [[raw/search-index-on-s3]], [[summary/search-index-on-s3]]*
*Last updated: 2026-10-24*
