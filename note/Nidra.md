# Nidra — Auto Ontology's engine on a graph instead of Postgres

Nidra is Sophia Labs' fork of NVIDIA Auto Ontology (fork point `cdaf61e`, 2026-11-03) with a single surgical change: Postgres is gone. The catalog, glossary, SQL attributes, analyses, zones, tags, rules and compilation history live as RDF in a **gardend** graph (the Sophia engine, PolyForm-licensed, reached over HTTP), and the vector index is gardend's own entity index with a locally-run multilingual-e5-small model — no embedding endpoint, no pgvector. Upstream's DAL golden captures (1,068 reads, originally captured for NVIDIA's own Neo4j→Postgres port) pass identically against the graph. It matters as a demonstration that a semantic-layer product can swap its entire storage substrate behind a stable read interface and prove the swap with the upstream vendor's own test data.

---

## Architecture

The engine keeps upstream's FastAPI surface (text-to-SQL agent, catalog APIs, MCP server) untouched; the change is entirely below the DAL:

- **`auto_ontology/rdf/vocab.py`** — the IRI scheme: a W3C Direct Mapping derived *mechanically from upstream's own SQLAlchemy metadata*. A row of table `T` with pk `id=X` becomes `<urn:ao:T/id=X>` typed `<urn:ao:T>`; column `c` is property `<urn:ao:T#c>`; foreign keys yield `<urn:ao:T#ref-c>` links. Because the mapping is derived from the metadata, "an upstream column added tomorrow is exported tomorrow with no code change."
- **`auto_ontology/rdf/patch.py`** — replaces DAL reads with RDF "ports" (`auto_ontology/dal_rdf/`, ~50 modules, one per DAL module, identical signatures and return shapes). It installs an **import hook** so ports are applied to targets already imported or right as they finish — `from auto_ontology.dal.terms import f` anywhere binds the port regardless of import order. `AO_RDF_STRICT=1` makes an unported read raise instead of silently falling back to Postgres.
- **`auto_ontology/rdf/writes.py`** (1,030 lines) — the single write seam, `commit(ops, witness, origin, txn)`, plus row primitives built on the Direct Mapping. Two transports via `NIDRA_WRITES`: `sparql` (default — ops rendered to ONE atomic SPARQL update in Oxigraph) or `commit` (ops appended to gardend's Emporium ledger with `wait=folded` read-your-writes after the 1s durability barrier).
- **`auto_ontology/vdb/gardend.py`** — the vector store as triples on the catalog rows themselves (`dcterms:description` carries the embed text, `urn:ao:vdb#metadata` the JSON metadata, `urn:ao:vdb#collection` the collection). Retrieval is gardend's `find` tool, which verbalizes each entity into one sentence and embeds it locally; the query passes text, not vectors, so the `EMBED_*` model triplet is simply not needed.
- **`Dockerfile.nidra`** — one container: engine, gardend binary, ONNX Runtime, the e5-small model, the Sophia UI. Install is a curl-sh one-liner with just an OpenRouter key.

## Key techniques

- **The oracle is upstream's own.** `auto_ontology/dal/tests/golden/dal_reads.json` was captured by NVIDIA for their migration; `test_golden.py` replays it against whichever backend is installed. The fork inherits a regression suite written by the project it forked.
- **Write grading, not just read grading.** `dev_tools/fixtures/write_diff.py` runs the same write (addressed by golden tokens like `<term:Film>`) against a throwaway Postgres copy (`CREATE DATABASE … TEMPLATE`) and a throwaway gardend graph, then diffs every DAL read of the resulting state. Deletes follow the SQLAlchemy metadata's `ON DELETE CASCADE`/`SET NULL` "the way Postgres would" — cascades become explicit ops because "the ledger knows no foreign keys."
- **The closure test.** `nidra/closure.sh` seeds NVIDIA's graph fixture with `POSTGRES_PORT=1` — the engine's own store unreachable — and replays the golden in strict mode. Postgres survives only as the *source warehouse being catalogued*, never as engine state.
- **Compensating writes as lineage.** The model import can't be one transaction (its SQL stage parses against what earlier stages wrote), so each stage commits after a preflight, and `writes.compensating()` records prior state of every touched subject; a failure issues one compensating commit, and "the ledger keeps the import and its undo as lineage."
- **An honest score-distance calibration.** `find`'s fused score caps at 0.70 for meaning-only hits, so naive `distance = 1 − score` made nonsense sit at 0.30 and `check_answerable` could never refuse. They measured 10 relevant vs 10 nonsense queries on a 274-entity fixture and mapped scores to distances so the thresholds (0.65/0.75) actually discriminate.
- **Radical demo honesty.** `demos/` records a 402-out-of-credit evening verbatim: certification writes 500, global search 500, an `COUNT(*)` validator deadlock reproduced with no model, readiness false-blockers — every demo graded against the source database, not the engine's self-report.

## Design decisions

- **Parity over improvement.** The demos' Part 1 is literally "Auto Ontology says it does X → Nidra does?" — the fork optimizes for provable behavioral equivalence, sacrificing features upstream gets from Next.js (no UI, no auth/RBAC in the engine, no OAuth MCP front door yet).
- **Mechanical derivation over hand-maintained schema.** Tying the ontology to SQLAlchemy metadata trades expressive freedom for zero-drift; the Direct Mapping is deliberately boring.
- **Ledger ownership has sharp edges.** In commit mode the fold only owns subjects it holds frames for: a value SPARQL-loaded beside a later commit survives the commit's `set` — so seeding must also go through commit (`ntriples_ops` turns the export into `set` ops). A subtle coupling between transport choice and data lifecycle.
- **Trade: a second, noncommercial dependency.** gardend is PolyForm Noncommercial — the Apache engine now requires a proprietary-licensed core for its storage, fine for a lab demo, a real constraint for enterprise adoption.

## Comparison notes

This is the first serious **fork-as-storage-port** in the wiki: where [[NVIDIA Auto Ontology]] puts the catalog in Postgres+pgvector, Nidra makes the same DAL a polymorphic interface over RDF, using upstream's golden captures as the acceptance test — an answer to [[Your Knowledge Graph Is Making Your Agent Dumber]]'s complaint that graphs of *code* add noise; here the graph *is* the product's native substrate, not a retrieval embellishment. Against [[Context Graphs]]' argument that "similarity is not relevance," Nidra's ledger-witnessed corrections are a concrete mechanism for recording *why* a catalog entry changed, with the undo as queryable lineage. And where [[Rip Vector Database]] argues separate vector stores are disappearing, Nidra goes further: the vector index hangs directly off the catalog rows as triples, so a row's embeddings die with its `retract` — no orphaned-index drift at all.

---
*Sources: [[raw/nidra]], [[summary/nidra]]*
*Last updated: 2026-11-06*
