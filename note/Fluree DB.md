# Fluree DB

Fluree DB is a ~50-crate Rust workspace for a temporal, verifiable RDF graph database that pitches itself at exactly the intersection this wiki keeps circling: knowledge-graph memory and tool surface for AI agents, without giving up database-engine performance. Its claim to attention is that it refuses the usual trade — it ships time travel, git-like branching, triple-level access control, RDFS/OWL reasoning, and integrated BM25/HNSW search, and then publishes reproducible benchmarks where it beats engines that have none of that (11.5× over QLever on DBLP-core SPARQLoscope, 43 ms geometric mean on the full 21.5B-triple Wikidata WGPB suite).

---

## Architecture

- **Workspace of ~50 crates** (`Cargo.toml`), layered roughly as: `fluree-graph-*` (IR, Turtle, JSON-LD parsing) → `fluree-db-core` (54K lines: datatypes, dictionaries, commits, content IDs, overlays) → `fluree-db-binary-index` + `fluree-db-indexer` (52K lines of storage engine) → `fluree-db-query` (196K lines: planner, operators, execution) → language frontends (`fluree-db-sparql` 33K lines, `fluree-db-cypher`, `fluree-db-graphql`) → surfaces (`fluree-db-server`, `fluree-db-cli`, `fluree-db-mcp`, `fluree-db-wasm`, `fluree-db-python`).
- **Content-addressed everything** (`fluree-db-core/src/content_id.rs`, `docs/design/content-id-and-contentstore.md`): commits, transactions, index roots, leaves and dictionary blobs are CIDv1 values (SHA-256 multihash, private multicodec range `0x300001`–`0x300007`). Identity is a content ID; location is a `ContentStore` trait — which is how memory, filesystem, S3+DynamoDB and IPFS backends stay interchangeable and why cached blobs never need invalidation.
- **Index format** (`docs/design/index-format.md`): four covering sort orders — SPOT, PSOT, POST, OPST — so nearly every triple pattern resolves to a contiguous range scan. The hierarchy is branch manifest (`.fbr`) → leaf file (`.fli`, ~10 leaflets) → leaflet (~25K rows) → per-column zstd blocks. Constant columns within a leaflet are omitted entirely (predicate-homogeneous POST leaflets store `p_const` once).
- **History is a sidecar, not a column**: per-leaf `FHS1` blobs hold 31-byte transition records, located via the leaf directory with `min_t`/`max_t` bounds, so a current-state query never touches or caches time-travel data. This is the cleanest trick in the design — temporal capability with zero cost on the hot path.
- **Replication**: `fluree-raft-core` is a deliberately application-agnostic Raft substrate ("nothing here knows what is being replicated"), meant to be instantiated once per replicated concern rather than one shared log — each group gets its own elections, log and snapshots.

## Key techniques

- **Integer-ID execution with late materialisation.** Everything — IRIs, strings, datatypes — is dictionary-encoded to u64/u32; joins, grouping and dedup run on integers, and values decode once, at projection, for surviving rows only (`fluree-db-query/src/binary_scan.rs` emits `EncodedSid`/`EncodedPid`/`EncodedLit`).
- **Directory-only answers.** Leaflet directory headers carry enough metadata (row counts, `first_key`/`last_key`, constant columns) that aggregates like `COUNT` on a single `(p,o)` group can be answered in O(leaflets) with zero decompression.
- **Correctness layered below optimisation.** Overlay merging, time travel and graph scoping are enforced in the scan layer (`BinaryCursor`), so no fast-path operator re-implements them. Each of the 16 fused operators has narrow admission criteria checked at runtime and a fallback to the generic plan; each has an env-var kill switch and `MustFire`/`MustNotFire` routing stamps in tests. The repo's own `CLAUDE.md` is strikingly honest about the epistemology: "two lanes agreeing is not evidence of correctness" — it cites issue #1700 where a shared-plan defect made both lanes return the same wrong answer, so differential testing needs an independent oracle.
- **Greedy planner, not DP.** Patterns are placed cheapest-eligible-first in three tiers (reducers → sources → expanders) using HLL-derived per-predicate statistics; the estimator handles known RDF cardinality traps (anchored transitive paths, DISTINCT subquery producers) explicitly because greedy placement offers no later correction.
- **The agent surface is engineered, not marketed** (`docs/ai/README.md`, `fluree-db-memory/`): "Agent JSON" is a query output envelope with a schema header, byte-budget truncation and a ready-to-run `resume` query; the MCP server exists in two forms (HTTP `/mcp` endpoint and stdio CLI with `docs` and `memory` toolsets); Fluree Memory stores coding-agent facts as structured triples in a local ledger, serialised as plain Turtle you can commit to git, with a `RecallEngine`, a `SecretDetector` that scrubs secrets before persistence, and an audit module.

## Design decisions

- **Speed and capability are not framed as a trade.** The performance doc opens by acknowledging that systems with time travel and reasoning are *assumed* to be slow, then argues the opposite with published numbers — the underlying bet is that columnar integer execution plus copy-on-write background indexing pays for the feature set.
- **Blob-store-first physical design.** Leaves bundle ~10 leaflets specifically to keep object counts low on S3-like stores while keeping fine-grained decompression inside leaflets — a cloud-economics decision baked into the on-disk format.
- **General mechanisms over shape recognition.** Fast paths are added only where the generic pipeline is orders of magnitude worse on a recurring shape, and every rewrite states its soundness condition (duplicate-insensitive aggregates, preserved multiplicity). This is the discipline most database projects claim and few document per-rewrite.
- **BUSL-1.1 with an Apache 2.0 change date** — source-available now, OSS later; you may build products on it but not resell it as a hosted database.
- **Honest limits sections.** Both the performance doc and the benchmark repo publish where the engine loses, which is rarer than the wins.

## Comparison notes

- Compared to [[Dolt]] — the other "git for data" database in this wiki — Fluree takes the versioning idea further (branch/rebase/merge on graph ledgers, content-addressed commits) but on an RDF data model rather than SQL tables; Dolt grafts git semantics onto MySQL, Fluree built the storage layer around immutability from day one.
- Against [[RIP Vector Database]]'s argument that ANN indexes should be secondary, not primary: Fluree agrees structurally — BM25 and HNSW live *inside* the query engine and participate in joins like any graph pattern, not as a bolted-on sidecar service.
- [[Your Knowledge Graph Is Making Your Agent Dumber]] found code knowledge graphs add noise agents can't filter; Fluree's bet is the complementary case — curated factual ledgers with typed edges, policies and provenance — and its Agent JSON byte-budgets show the same lesson applied at the output layer: the graph must respect the token budget, not just the retrieval.
- [[MotherDuck Getting Started]] pitches agents as a first-class database customer via an MCP endpoint; Fluree makes the same pitch but goes further with a purpose-built memory product (`fluree mcp serve` + Fluree Memory) — the two make 2026's clearest pair of examples that database vendors now design for LLM consumers, not just human ones.
- See also [[Context Graphs]] and [[Zero-Mem — Zero-Token Memory Operations]] on structured vs. vector agent memory: Fluree Memory is firmly in the structured camp, and its "recall stays small" tuning is the same token-budget argument.

---
*Sources: [[raw/db]], [[summary/db]]*
*Last updated: 2026-10-20*
