# NVIDIA Auto Ontology — the ontology as agent context layer

NVIDIA's open-source (~120K-line) answer to the reliability problem of agentic analytics: a system that compiles enterprise metadata into a certified semantic/ontology layer and serves it to agents over MCP, API, and chat. Its thesis is that the bottleneck for agents querying data is not SQL generation but grounded meaning — where data lives, what a metric means, whether to use it, and provenance — so it builds that layer automatically and lets certified corrections accumulate back into it.

---

## Architecture

Four services: Next.js 16 frontend (auth gateway, Better Auth, Prisma), FastAPI backend hosting the LangGraph text-to-SQL pipeline, a separate ingestion worker, and Postgres+pgvector holding both the app catalog and the vector store. Optional Vault for credentials; Helm or Docker Compose deployment.

Three subsystems do the real work:

- **`auto_ontology/ingestion_service/`** — schedulers that ingest schema metadata and run PII detection; a `semantic_scheduler.py` compiles the ontology per connection on a 24h interval, with an explicit dependency so the first compilation pass waits for the ingest pass (otherwise it silently produces zero Terms against an empty catalog — the kind of race comment you rarely see documented).
- **`auto_ontology/semantic/`** — the compilation pipeline: term extraction, FK suggestion (`semantic_fk.py`, `fk_suggester.py`), SQL attribute suggestion, bridge-table detection, embedding. `pipeline.py` runs 3 worker threads over tables with an `_OrderedCommitQueue` — LLM extraction is parallel but commits are serialized in table order, and a `skip()` primitive lets a worker give up its turn *out of order* rather than block the pool.
- **`auto_ontology/retrieval/text_to_sql/`** — a ~25-node LangGraph (`text_to_sql_graph.py`): intent classification → question extraction → candidate retrieval → evidence refinement → SQL construction from candidates → parse/intent validation → execution, with an `EmptyResultValueRepairAgent`, an `EmptyLikeResultCheckAgent`, a merged proactive precheck (literal/join-path/JSONB checks in one node to save round-trips), and an `unconstructable_sql_response` terminus after 7 failed reconstructions.

Exposure is threefold: the MCP server (`mcp/auto_ontology_mcp/`), the `/api/chat/completions` SSE gateway, and three agent skills (`nvidia-ontology-setup/-management/-query`) plus an AGENTS.md/CLAUDE.md split inside the repo itself.

## Key techniques

- **Curated MCP tool surface as prompt engineering.** `mcp/auto_ontology_mcp/tools.py` allows only a handful of the 87 OpenAPI operations. The reasoning is explicit and worth quoting: agent tool-selection degrades well before 87 tools, and "a tool description is a prompt." Most strikingly, there is deliberately *no tool for browsing schemas or columns* — because such a tool invites the agent to bypass the semantic layer and reason about raw tables, defeating the whole product. Tool names/descriptions are hand-overridden; CI fails if the spec drifts.
- **Soft checks with escape hatches.** Post-construction validation checks (join-path, value, JSONB, empty-LIKE) send SQL back for reconstruction, but `_make_soft_check_router` stops blocking once `INTENT_VALIDATION_SKIPPED_AFTER` failed attempts accumulate — otherwise a persistently failing check loops until the graph's global recursion limit kills the run. The hard cap (7) applies only to reconstruction itself.
- **Structured precedent retrieval.** The state carries `sql_examples` (question/SQL pairs retrieved as structural precedent, "advisory... they inform query shape only — never identifiers or filter values") separately from `evidence` and `value_anchors`. A careful separation of what retrieved context is allowed to influence.
- **Conversation repair, not raw history.** Follow-ups ("what about August?") are rewritten into standalone questions using only the five most recent *completed* turns, bounded to 12K characters; internal agent thoughts, charts, errors, and raw result sets are excluded from context reuse.
- **Config-as-triplet.** Every model role (reasoning, non-reasoning, embed, rerank) is a `<PREFIX>_ENDPOINT/_API_KEY/_MODEL` triplet with fallback to `DEFAULT_MODELS_*` — a small, clean pattern for multi-model deployments.
- **Graph topology as deployment config.** The KumoRFM prediction branch and the combined precheck are wired into the LangGraph *at startup only* if their env flags are set — so an unconfigured deployment doesn't even have the nodes.

## Design decisions

- **Correctness over capability in the tool surface** — fewer tools, semantic-only access, read-only MCP. Sacrifices agent flexibility for grounded answers.
- **Automation with certification, not autonomy.** The ontology is *generated* from metadata, but the "Learn" loop only folds back *certified* corrections — the human certifies, the machine maintains. Same bet as Netflix's entropy-ladder trick in [[Ontology-Driven Observability at Netflix]]: use LLMs to build the graph, but make the ingest path deterministic.
- **Trust boundary made explicit.** FastAPI trusts the `x-auto-ontology-user-id` header forwarded by the Next.js gateway and must never be exposed publicly — a documented, deliberate simplification rather than an accident. API tokens carry the owner's role, are stored only as SHA-256 hashes, and cannot mint successors.
- **Plaintext credentials by default.** UI-added connections store passwords unencrypted in Postgres unless Vault is fully configured; partial Vault config is ignored with a warning. Pragmatic, but a soft spot.

## Comparison notes

- **[[Vedana — Domain Models as Agent Context]]** is the closest sibling: both ground agent answers in a curated semantic model rather than raw chunks, and both report that exact, traceable answers beat vector-search guessing. Auto Ontology automates the model-building Vedana does by hand — the exact "who maintains it" wall Vedana's talk ends at is what Auto Ontology's compilation pipeline and certification loop are built to attack.
- **[[Databricks Genie Spaces for SQL Analysts]]** solves the same governed-NL-querying problem inside one vendor's platform (curated views, certified metrics, example pairs as regression tests). Auto Ontology is the open, cross-database equivalent — Databricks, Snowflake, DuckDB, Trino, ClickHouse connectors — with the ontology layer stored next to the catalog instead of inside the warehouse.
- **[[Context Graphs]]** argues for typed structure over vector similarity as the agent-retrieval primitive; Auto Ontology is a production instantiation of that argument for the data domain, with pgvector used for candidate *recall* but certified terms and SQL attributes carrying the *meaning*.
- **[[Stateless MCP]]**'s minimal-MCP thesis shows up here inverted but consonant: the MCP server holds no credentials, no DB connections, no model config — it is a thin authenticated HTTP client of the public API, so it runs on a laptop far from the deployment.

---
*Sources: [[raw/auto-ontology]], [[summary/auto-ontology]]*
*Last updated: 2026-11-03*
