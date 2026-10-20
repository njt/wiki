---
url: https://github.com/fluree/db
title: "Fluree DB — A graph database for data that matters"
author: Fluree
date_fetched: 2026-10-20
date_published: 2026-10-20
topics:
  - databases-and-data
  - agent-memory-and-context
---

Fluree DB is a Rust workspace (~50 crates) implementing a temporal, content-addressed RDF graph database: SPARQL 1.1/1.2, JSON-LD, openCypher and GraphQL over one engine, with built-in time travel, git-like branching, triple-level access control, RDFS/OWL 2 RL reasoning, and integrated BM25 + HNSW search. The README stakes its claim on benchmarks: on the 21.5-billion-triple Wikidata dump it runs all 850 WGPB graph-pattern queries at a 43 ms geometric mean, and on DBLP-core SPARQLoscope it reports 11.5× over QLever — while carrying features the specialist engines lack.

The performance story rests on seven choices documented in `docs/design/performance.md`: dictionary-encoded u64 execution, four covering index permutations (SPOT/PSOT/POST/OPST) with per-column zstd-compressed blocks inside leaflets, aggregates answerable from leaflet directory headers in O(leaflets), an HLL-statistics-driven cost model, specialised join operators, 16 fused "fast-path" operators each with a correctness fallback, and writes that commit to an in-memory overlay while background indexing runs copy-on-write. History lives in a per-leaf sidecar object, so time travel costs current-state queries nothing.

Storage is identity-separated from location: every artifact (commit, index root, leaf, dictionary blob) is a CIDv1 content ID (SHA-256, multicodec private range) fetched through a `ContentStore` trait — which is what makes IPFS, S3+DynamoDB, and file backends interchangeable and blobs cacheable without invalidation. Replication is built on a generic Raft substrate (`fluree-raft-core`) instantiated once per replicated concern.

The agent pitch is concrete, not decorative: the CLI ships `fluree mcp serve` (stdio MCP with docs and memory toolsets), a server `/mcp` endpoint returning token-budgeted "Agent JSON", and **Fluree Memory** — persistent, git-committable project memory for Claude Code/Cursor stored as structured facts in a local ledger with ranked recall and secret detection. Licensed under BUSL-1.1 with a change date to Apache 2.0.
