---
url: https://github.com/ptaranat/zeromem
title: zeromem — Zero-token memory operations for LLM agents
author: Panat Taranat
date_fetched: 2026-08-06
source: https://arxiv.org/abs/2607.29377 (Zero-Mem paper by Xiao et al.)
---

# zeromem (summary)

A Rust implementation of Zero-Mem (Xiao et al., arXiv:2607.29377): agent memory where every operation outside final question answering makes zero LLM calls and consumes zero tokens. Raw conversation turns are the source of record; retrieval is structured search over them, not generated summaries.

Ships as a Rust crate, a `zm` CLI, a Python module (via PyO3), and a memory provider plugin for Hermes Agent.

## How it works

Ingest indexes each turn three ways: an entity-context graph (heuristic NER, co-occurrence edges), a temporal hierarchy (turns → windows → episodes, split on session change, time gap, or topic drift), and BM25 plus dense embeddings. A query builds a deterministic profile (subject, keywords, answer type, temporal cues, session boundary), routes to a primary view (entity-relational vs. temporal-local), runs both retrievals, fuses with min-max normalization, closes over graph bridges and local neighbors, and calibrates the evidence set. Optionally the host can calibrate the reader's answer against typed evidence candidates.

Persistence is a single SQLite file (turns plus embedding cache). Graph, hierarchy, and BM25 rebuild from turns on open — the store cannot drift from the indexes.

Deviations from the paper: spaCy NER replaced with a heuristic extractor behind a trait; BGE-M3 replaced with bge-small-en-v1.5 via fastembed-rs (a deterministic hash embedder is the offline fallback). Both are swappable via the `Embedder` and `EntityExtractor` traits.

The core is ~2,700 lines of Rust across 17 source files. Default parameters follow the paper: gamma 0.6, rho 0.6, top-5 evidence.
