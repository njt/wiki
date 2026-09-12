---
url: https://github.com/alash3al/stash
title: "Stash: Persistent Memory for AI Agents"
author: alash3al (Mohamed Al Ashaal)
date_fetched: 2026-05-22
date_published: 2026-05
topics:
  - agent-memory-and-context
---

# Stash: Persistent Memory for AI Agents

## Overview

Stash is an open-source, self-hosted persistent memory system for AI agents. It gives LLMs persistent memory across sessions via the MCP (Model Context Protocol). Built in Go with PostgreSQL + pgvector, it runs as a single binary with an 8-stage cognitive consolidation pipeline that transforms raw observations (episodes) into structured facts, relationships, patterns, causal links, and more — then decays stale beliefs over time.

GitHub: https://github.com/alash3al/stash
License: Apache 2.0
Language: Go 1.25.5
Dependencies: PostgreSQL + pgvector, OpenAI-compatible API (for embeddings and reasoning)

## Architecture

### Layered Design

```
CLI (urfave/cli v3) → Brain (business logic) → Reasoner/Embedder (AI services) → PostgreSQL + pgvector
```

**Entry point**: `cmd/cli/main.go` — a single CLI binary with subcommands for serve, remember, recall, forget, consolidate, facts, context, contradictions, causal, hypothesis, goal, failure, mcp, namespace.

**Core struct**: `Brain` (internal/brain/brain.go:78) wraps four dependencies:
- `pgxpool.Pool` — PostgreSQL connection pool
- `embedder.Embedder` — embedding generation interface
- `reasoner.Reasoner` — LLM reasoning interface  
- `queries.Queries` — sqltmpl-based SQL template engine

**Bootstrap**: `internal/bootstrap/bootstrap.go` wires everything together — loads config from `.env`, opens the database (with goose migrations + HNSW index creation), builds the embedder (wrapped in a cache), builds the reasoner, and creates the Brain.

### Data Model (internal/models/models.go)

The system has 16 domain types:

1. **Namespace** — hierarchical memory scope (e.g., `/self/capabilities`, `/projects/stash`)
2. **Episode** — immutable, append-only raw observation with vector embedding
3. **Fact** — consolidated belief derived from episodes, with structured fields (entity, property, value) and confidence
4. **FactSource** — links a fact to its source episodes
5. **Relationship** — extracted entity edge (from_entity, relation_type, to_entity)
6. **Pattern** — abstraction over facts and relationships with coherence score
7. **Contradiction** — conflict between two facts about the same entity+property
8. **CausalLink** — cause-effect relationship between two facts
9. **Hypothesis** — belief held with uncertainty + verification plan, lifecycle: proposed → testing → confirmed/rejected
10. **Goal** — intended outcome with sub-goal hierarchy, status: active → completed/abandoned
11. **Failure** — what didn't work, why, and lesson learned
12. **Context** — short-lived working focus for a namespace (expires)
13. **ConsolidationProgress** — per-namespace checkpoint tracking for each pipeline stage
14. **Setting** — key-value store for operational state
15. **EmbeddingCache** — stored embeddings by text hash + model
16. **RecallResult** — unified search result from episodes + facts

### 8-Stage Consolidation Pipeline (internal/brain/consolidate.go)

The pipeline runs per-namespace and processes only new data since the last run (checkpoint-based incremental processing):

1. **Episodes → Facts** (consolidateEpisodesToFacts): Groups episodes by vector similarity (greedy clustering), calls the reasoner to synthesize a structured fact, embeds it, checks for duplicates, inserts with fact_sources linkage. Also runs Stage 4 (contradiction detection) inline.

2. **Facts → Relationships** (consolidateFactsToRelationships): For each new fact, calls the reasoner to extract entity relationships (from_entity --relation_type--> to_entity).

3.5. **Facts → Causal Links** (consolidateFactsToCausalLinks): Feeds batch of facts to the reasoner to extract cause-effect pairs.

6. **Goal Progress Inference** (consolidateGoalProgress): Assesses whether recent facts indicate progress, completion, or contradiction of active goals. Annotates goals with progress notes.

7. **Failure Pattern Detection** (consolidateFailurePatterns): Checks if recent episodes repeat past failures, and extracts higher-order failure patterns as new facts.

3. **Facts + Relationships → Patterns** (consolidateToPatterns): Extracts abstract patterns spanning multiple facts and relationships with coherence scoring.

8. **Hypothesis Evidence Scanning** (consolidateHypothesisEvidence): Tests open hypotheses against new facts. Auto-confirms hypotheses when supporting evidence confidence exceeds threshold (default 0.9). Auto-rejects when contradicting evidence exceeds threshold. Updates confidence for supported/weakened hypotheses.

5. **Confidence Decay** (DecayConfidence): Pure SQL — multiplies fact confidence by DecayFactor (default 0.95) for facts not updated within the configurable window (default 7 days). Facts below ExpiryThreshold (default 0.1) are soft-deleted.

### Key Implementation Details

**Greedy Vector Clustering** (consolidate.go:314): Episodes are clustered by cosine similarity using a simple greedy algorithm. Each unclustered episode becomes a seed; all other unclustered episodes within the similarity threshold (default 0.85) join its cluster. This is O(n²) but for batch sizes of 100 it's negligible vs. the LLM API call cost.

**Contradiction Detection** (contradiction.go:18): When a new structured fact is inserted with (entity, property, value), the system queries for existing facts with the same entity+property but different value. Each pair is sent to the reasoner for classification:
- **Replacement** (confidence ≥ 0.9): Auto-supersede the old fact (set valid_until)
- **Replacement** (confidence < 0.9): Record as contradiction for human review
- **Contradiction**: Record as contradiction for human review
- **Compatible**: Both values can coexist, no action

**Checkpoint Safety** (throughout consolidate.go): Progress checkpoints are only advanced if no errors occurred during the stage. This prevents data loss — if a stage fails, it will re-process the same data on the next run. The final save even uses `context.Background()` if the original context was cancelled.

**Embedding Cache** (embedder/cache.go): SHA-256 hash of text + model name as cache key. In-flight request deduplication via sync.Map — concurrent requests for the same text wait on a sync.WaitGroup rather than making duplicate API calls.

**Causal Chain Tracing** (causal.go:116): Uses PostgreSQL recursive CTEs for bounded-depth traversal of cause→effect chains. Supports forward (what did this cause?) and backward (what caused this?) directions.

**Namespace Resolution** (brain.go:162): Hierarchical paths — `/projects/stash` matches that exact namespace AND all descendants (`/projects/stash/backend`). Root `/` matches everything. Used for both search and write operations.

### MCP Integration

The MCP server (cmd/cli/mcp.go) exposes 27 tools via the mark3labs/mcp-go library over SSE and stdio transports. The prompts template (mcp_prompts.tmpl, ~1000 lines) is a detailed behavioral contract for the AI agent — it defines:
- Prime Directive for memory usage
- Proactivity clause (store useful info without being asked)
- Tool decision tree
- Session protocol (init → recall → work → consolidate → context handoff)
- Namespace rules
- Self-model (/self/capabilities, /self/limits, /self/preferences)
- Per-tool detailed descriptions with good/bad examples and "cost of not calling"

### Dependencies

- `pgvector/pgvector-go` — PostgreSQL vector operations
- `mark3labs/mcp-go` — MCP protocol implementation
- `openai/openai-go` — OpenAI-compatible API client (works with any endpoint: OpenAI, OpenRouter, Ollama, vLLM)
- `pressly/goose` — Database migrations
- `jackc/pgx` — PostgreSQL driver
- `alash3al/sqltmpl` — SQL template engine (by the same author)
- `urfave/cli/v3` — CLI framework
- `prometheus/client_golang` — Metrics

## Design Analysis

### What Makes This Different

Unlike simple vector DBs (Pinecone, Chroma) or memory plugins (Mem0, MemGPT), Stash models the full cognitive pipeline:
- **Observation → Belief**: Episodes are raw; facts are synthesized truths
- **Belief → Structure**: Entity/property/value extraction enables contradiction detection
- **Structure → Abstraction**: Patterns emerge from multiple facts and relationships
- **Causality**: Explicit cause-effect links, not just correlation
- **Uncertainty Management**: Hypotheses with verification plans, confidence scoring
- **Forgetting**: Controlled decay of unconfirmed beliefs
- **Failure Learning**: Explicit failure recording with detection of repeated patterns

### Design Trade-offs

**Optimized for**: Correctness and safety. Checkpoints prevent data loss. Retry loops prevent hallucinated facts. Grounding validation prevents invented entities. Anti-verbatim checks prevent copying source text as facts.

**Sacrificed**: Horizontal scalability. Single PostgreSQL instance. Batch sizes capped at 100. No distributed coordination.

**Vendor choice**: OpenAI-compatible API only for embedder and reasoner — but the interface design would allow alternative implementations. The system prompt is locked to JSON extraction patterns that assume an OpenAI-compatible chat completions endpoint.

### Comparison to Related Projects

- **vs. Claude-Mem**: Claude-Mem captures session transcripts and compresses them. Stash extracts structured knowledge from observations — facts, relationships, causal links — rather than storing compressed transcripts.
- **vs. GraphRAG**: GraphRAG builds knowledge graphs from documents for RAG. Stash builds them from agent observations, with contradiction detection, confidence decay, and hypothesis testing that GraphRAG doesn't have.
- **vs. NornicDB**: Both use graph + vector + temporal storage with decay. NornicDB is a general-purpose database; Stash is an agent-memory appliance with an MCP interface and cognitive pipeline.
- **vs. Mem0**: Mem0 is a hosted memory API. Stash is self-hosted, MCP-native, and has a much richer cognitive model (goals, hypotheses, failures, causal links, contradictions).

### Weaknesses

- **Single-writer**: The checkpoint system assumes one consolidation process per namespace. Multiple concurrent consolidators would duplicate work.
- **Greedy clustering**: Simple but can produce suboptimal clusters. An episode at the boundary of two clusters always joins the first cluster found, even if it fits better with the second.
- **No streaming**: The recall system returns batch results. No incremental or streaming retrieval.
- **OpenAI coupling**: Despite interface abstractions, the reasoner prompts are tightly coupled to OpenAI-compatible chat completion format. Swapping to Claude API or Gemini would require prompt restructuring.
- **Context window pressure**: The consolidate stage sends batches of 30-50 facts to the LLM. For namespaces with thousands of facts, the batch window limits pattern detection to local relationships.
