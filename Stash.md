# Stash

Open-source persistent memory for AI agents. A single Go binary backed by PostgreSQL+pgvector that gives LLMs durable memory across sessions via MCP, with an 8-stage cognitive consolidation pipeline that transforms raw observations into structured knowledge — facts, relationships, causal links, patterns — and decays stale beliefs over time.

---

## Architecture

Stash is a layered Go application (`cmd/cli/main.go`) structured as:

```
CLI (urfave/cli v3) → Brain → Reasoner/Embedder → PostgreSQL + pgvector
```

The **Brain** struct (`internal/brain/brain.go:78`) is the central orchestrator, wrapping a pgxpool, an embedder, a reasoner, and SQL query templates. Every operation flows through the Brain.

**Bootstrap** (`internal/bootstrap/bootstrap.go`) wires everything: loads `.env` config, opens the database (running goose migrations + creating HNSW indexes on `episodes.embedding` and `facts.embedding`), wraps the embedder in a SHA-256 hash-based cache, and builds the reasoner.

### Data Model

The system defines 16 domain types (`internal/models/models.go`):

- **Episode** — immutable, append-only raw observation with vector embedding
- **Fact** — consolidated belief with entity/property/value structured fields and confidence score
- **Relationship** — extracted entity edge (from_entity --relation_type--> to_entity)
- **Pattern** — abstraction over multiple facts and relationships with coherence scoring
- **CausalLink** — cause-effect relationship between two facts
- **Contradiction** — conflict between two facts about the same (entity, property)
- **Hypothesis** — belief with uncertainty + verification plan; lifecycle: proposed → testing → confirmed/rejected
- **Goal** — intended outcome with sub-goal hierarchy; status: active → completed/abandoned
- **Failure** — what was attempted, why it failed, and the lesson
- **Context** — short-lived working focus for a namespace (expires after TTL)
- **Namespace** — hierarchical memory scope (e.g., `/projects/stash`, `/self/capabilities`)
- **ConsolidationProgress** — per-namespace checkpoint tracking for incremental processing

### The 8-Stage Consolidation Pipeline

The consolidation pipeline (`internal/brain/consolidate.go`) runs per-namespace and processes only new data since the last run via checkpoint IDs. The stages execute in this order:

1. **Episodes → Facts** — greedy cosine-similarity clustering groups related episodes, then the reasoner synthesizes a structured fact with entity/property/value extraction. Duplicate facts are detected by vector similarity and deduplicated. Also runs contradiction detection inline.

2. **Facts → Relationships** — for each new fact with structured fields, the reasoner extracts entity relationship triples.

3.5. **Facts → Causal Links** — feeds a batch of facts to the reasoner to extract cause-effect pairs (`internal/brain/causal.go`).

6. **Goal Progress Inference** — assesses whether recent facts indicate progress toward, completion of, or contradiction of active goals (`internal/brain/consolidate_goal.go`).

7. **Failure Pattern Detection** — checks if recent episodes repeat past failures, and extracts higher-order failure patterns as new facts (`internal/brain/consolidate_failure.go`).

3. **Facts + Relationships → Patterns** — extracts abstract patterns spanning multiple facts and relationships with coherence scoring.

8. **Hypothesis Evidence Scanning** — tests open hypotheses against new facts. Auto-confirms when supporting evidence confidence ≥ 0.9. Auto-rejects when contradicting evidence ≥ 0.9 (`internal/brain/consolidate_hypothesis.go`).

5. **Confidence Decay** — pure SQL: multiplies fact confidence by a decay factor (default 0.95) for facts not updated within the window (default 7 days). Facts below the expiry threshold (default 0.1) are soft-deleted (`internal/brain/decay.go`).

The pipeline numbering (1,2,3.5,6,7,3,8,5) reflects the actual execution order — causal links come before patterns, and decay runs last.

### MCP Integration

The MCP server (`cmd/cli/mcp.go`) exposes 27 tools via SSE and stdio transports. The prompts template (`cmd/cli/mcp_prompts.tmpl`, ~1000 lines) is a detailed behavioral contract for AI agents — it defines session protocol (init → recall → work → consolidate → context handoff), namespace rules, a self-model under `/self`, and per-tool detailed guidance with explicit "cost of not calling" admonitions.

Default namespace scaffold created by the `init` tool:
- `/self` — agent's self-knowledge root
- `/self/capabilities` — what the agent can do well
- `/self/limits` — what the agent struggles with
- `/self/preferences` — how the agent works best

## Key Techniques

### Greedy Vector Clustering

Episodes are clustered by cosine similarity using a simple greedy algorithm (`consolidate.go:314`). Each unclustered episode becomes a seed; all other unclustered episodes within the similarity threshold (default 0.85) join its cluster. O(n²) but negligible vs. LLM API call cost at batch sizes of 100. This is simpler than k-means or DBSCAN and doesn't require pre-specifying the number of clusters.

### Defensive LLM Reasoning

The reasoner (`internal/reasoner/openai.go`) uses aggressive prompt engineering:
- **Strict system prompt**: "You are a strict information extraction engine. Extract ONLY what is explicitly stated. Never guess. Never approximate."
- **2-attempt retry loop**: On parse failure or validation failure, the system prompt is appended with a retry warning and the specific error
- **Grounding validation**: `validateFactGrounding` checks the summary isn't a verbatim copy of any source text (tokenizing and checking >80% word overlap triggers retry)
- **ID validation**: Extracted source IDs (fact IDs, relationship IDs) are checked against the actual provided IDs — hallucinated IDs trigger retry

### Contradiction Detection with Auto-Resolution

When a new fact is inserted with (entity, property, value), the system queries for existing facts with the same entity+property but different value (`internal/brain/contradiction.go:18`). Each pair is sent to the reasoner for classification:
- **Replacement** (confidence ≥ 0.9): auto-supersede the old fact by setting `valid_until`
- **Replacement** (confidence < 0.9): record as unresolved contradiction for human review
- **Contradiction**: record as unresolved
- **Compatible**: both values can coexist, no action

### Checkpoint Safety

Progress checkpoints are only advanced if no errors occurred during that stage. If a stage fails, it re-processes the same data on the next run. The final checkpoint save even uses `context.Background()` if the original context was cancelled — preventing partial-state loss.

### Embedding Cache with Request Deduplication

`internal/embedder/cache.go` uses SHA-256 hash + model name as cache key, backed by the `embedding_cache` PostgreSQL table. In-flight request deduplication via `sync.Map` — concurrent requests for the same text wait on a `sync.WaitGroup` rather than making duplicate API calls.

### Hierarchical Namespace Resolution

Paths like `/projects/stash` match that exact namespace AND all descendants (`/projects/stash/backend`). Root `/` matches all namespaces. `resolveNamespaceIDWithDescendants` (`brain.go:200`) uses `LIKE` pattern matching for descendant resolution.

### Recursive CTE Causal Tracing

`TraceCausalChain` (`causal.go:116`) uses PostgreSQL recursive CTEs to walk cause→effect or effect→cause chains with bounded depth. Supports forward ("what did this cause?") and backward ("what caused this?") traversal.

## Design Decisions

### Optimized for Correctness, Not Throughput

Every pipeline stage has error accumulation rather than fail-fast. Checkpoints are only advanced on success. Batch sizes are capped at 30-100. This means consolidation is slow but safe — it prioritizes never losing data over processing speed.

### Self-Hosted by Default

Unlike Mem0 (hosted API), Stash is a single binary you run yourself. Docker Compose brings up Postgres + pgvector + the Stash binary. The trade-off is operational burden for data sovereignty.

### MCP as the Only Interface

There's no REST API, no gRPC, no SDK. MCP is the sole integration surface. This is a strong bet on MCP as the standard agent protocol. The HTTP server only serves metrics and health checks.

### Cognitive Metaphor Over Engineering Pragmatism

Stash explicitly models biological memory concepts: episodic memory (raw observations), semantic memory (consolidated facts), consolidation (sleep-like batch processing), decay (forgetting), and contradiction resolution. This is opinionated — it adds complexity over a simple vector store, but provides richer retrieval patterns (causal tracing, contradiction detection, goal tracking).

### All Soft-Delete, No Hard-Delete (by default)

Episodes, facts, relationships, patterns, hypotheses, and goals all use `deleted_at` soft-delete. The `forget` command is a soft-delete. Only `purge episode` does a hard delete. This means the system never truly forgets — it only hides. Good for audit, bad for GDPR.

### Confidence as a First-Class Concept

Every fact has a confidence score. Confidence is calculated from observation count (Laplace-like smoothing: `n/(n+2)`) with a boost for structured fields. Confidence decays over time. Low-confidence facts are soft-deleted. This means the system has an opinion about what it knows vs. what it merely suspects — unlike most memory systems where everything is equally "true."

## Comparison Notes

- **vs. [[Claude-Mem]]**: Claude-Mem captures and compresses session transcripts. Stash extracts structured knowledge — facts with entity/property/value, relationships, and causal links — rather than storing compressed transcripts. Claude-Mem is a recording; Stash is a mind.

- **vs. [[GraphRAG]]**: Both build knowledge graphs. GraphRAG builds them from documents for RAG; Stash builds them from agent observations, with contradiction detection, confidence decay, and hypothesis testing that GraphRAG doesn't have.

- **vs. [[NornicDB]]**: Both use graph + vector + temporal storage with decay. NornicDB is a general-purpose database; Stash is an agent-memory appliance with an MCP interface and cognitive pipeline.

- **vs. [[Memory Mechanism]]**: xAI's five-type taxonomy (semantic, episodic, procedural, etc.) maps cleanly onto Stash's data model — episodes (episodic), facts (semantic), patterns/goals (procedural). Stash is a working implementation of several of those memory types in a single system.

- **vs. [[How AI Agent Memory Works]]**: Cobanov describes HyDE and RRF as retrieval techniques. Stash doesn't use these — it relies on direct vector similarity with pgvector's `<=>` cosine distance operator, prioritizing simplicity over retrieval optimization.

- **vs. [[Claude Memory Extractor Research]]**: The 15-agent experiment found agents are overconfident on ambiguous cases. Stash addresses this with structured confidence scoring, a dedicated hypothesis type with verification plans, and automatic evidence re-scanning during consolidation — a partial mitigation, though the underlying problem of LLM overconfidence remains.

## Weaknesses

- **Single-writer consolidation**: The checkpoint system assumes one consolidator per namespace. Multiple concurrent runs would duplicate work.
- **Greedy clustering quality**: Boundary episodes always join the first matching cluster, even if they fit better with the second.
- **No streaming retrieval**: `Recall` returns batch results with no incremental or cursor-based retrieval.
- **OpenAI coupling in practice**: Despite interface abstractions, the reasoner's 2-attempt retry + structured JSON extraction pattern is tightly coupled to OpenAI-compatible chat completion format.
- **Context window limits on pattern detection**: Consolidation sends batches of 30-50 items to the LLM. Cross-batch patterns are invisible unless they re-emerge in later batches.

#tool #project #agent-memory

---

*Sources: [[raw/stash]]*
*Last updated: 2026-05-22*
