---
url: https://github.com/sophia-labs/nidra
title: "Nidra — NVIDIA Auto Ontology on the Sophia graph engine"
author: Sophia Labs
date_fetched: 2026-11-06
date_published: 2026-11-03
topics:
  - coding-agents-and-frameworks
  - databases-and-data
---

Nidra is Sophia Labs' fork of NVIDIA Auto Ontology (fork point `cdaf61e`, modifications from 2026-11-03, Apache-2.0) that removes Postgres entirely: the catalog, glossary, SQL attributes, zones, tags, rules and compilation history live as RDF in a **gardend** graph — the Sophia engine — and the vector index is gardend's own entity index, with a local multilingual-e5-small embedding model baked into the image. No embedding endpoint is needed at all.

The trick that makes it credible is an **oracle-based port**: the catalog's SQLAlchemy metadata is mechanically translated into a W3C Direct Mapping IRI scheme (`urn:ao:<table>/<pk>=<v>`), DAL reads are replaced by SPARQL "ports" with identical signatures installed via an import hook, and the whole thing is graded against NVIDIA's own golden captures (1,068 reads from upstream's Neo4j→Postgres port) plus a write-diff harness that applies each write to Postgres and gardend and diffs every DAL read. The `nidra/closure.sh` test proves the engine runs with its Postgres store unreachable.

Writes go through a single seam (`rdf/writes.py commit()`), transportable as either one atomic SPARQL update or an append to gardend's Emporium ledger with `wait=folded` read-your-writes; a failed multi-stage model import is undone by a recorded compensating commit that stays in the ledger as lineage. The repo ships an unusually honest `demos/` suite: 25+ scripts graded against the source databases themselves, with recorded outputs marking partial passes (broken search, 402-killed runs, an unanswerable-question detector that misses nonsense) rather than rounding up.
