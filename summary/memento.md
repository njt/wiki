---
url: https://github.com/latentsignal-org/memento
title: Memento
author: latentsignal-org (Jesse Vincent / Prime Radiant)
date_fetched: 2026-06-15
date_published: 2026-06
topics:
  - personal-agents
  - agent-architecture
---

# Memento — Raw Analysis

Memento turns years of email into source-attributed living documents. It is a local-first knowledge layer over a `msgvault` email archive, built for the long-term signal that ordinary inboxes and keyword search leave buried. Memento is not an email client and does not rebuild your inbox. It organizes long-term email history into five memory surfaces: People (relationship wikis), Projects (bounded narratives), Newsletters (coverage summaries), Concepts (user-declared evergreen topics), and Home (overview + "Ask Memento" chat).

## Architecture

- **Single binary, single origin**: Go binary embeds statically exported Next.js frontend, serves both UI and API on `127.0.0.1:8787`
- **Dual database**: msgvault SQLite read-only (archive substrate); Memento writes to `memento_*` tables
- **Five agents running in Go**: collector, project_compile, concept_compile, person_enrich, dashboard — all live in `backend/internal/agentrunner`
- **Rollup pattern**: materialized `memento_*_report` tables for fast index reads, rebuilt via `./memento refresh`
- **Frontend**: Next.js 16 static export (`output: "export"`), React 19, shadcn/ui, @base-ui/react
- **Backend Go deps**: minimal — `modernc.org/sqlite`, `golang.org/x/sync`, `github.com/google/uuid`
- **No Node.js at runtime** — the Go binary serves the static export directly

## Pipeline Summary

| Dimension   | Deterministic                           | LLM (agent)                        |
|-------------|-----------------------------------------|------------------------------------|
| People      | Person resolution, classification, rollup | Person enrich (facets, narrative) |
| Projects    | CRUD, message attachment, bundle, rollup | Project compile (4-section narrative) |
| Newsletters | Source detection, rollup                | One-shot coverage summary          |
| Concepts    | CRUD, message attachment, bundle, rollup | Concept compile (3-section narrative) |
| Dashboard   | Composes rollups                        | Ask Memento router agent           |

## Agent Runtime

The agent loop is a 20-step (configurable) model/tool loop with:
- **Completion contracts**: each agent type MUST call specific write tools or the run fails
- **Repair turns**: one bounded repair attempt when required outcomes are missing
- **Outcome tracking**: `OutcomeRequirement` structs with `ToolName`, `ArgEquals`, `AnyOfGroup`, `RequiredCount`
- **Tool locking**: read-only tools execute in parallel (4-8 concurrent); mutating/human-waiting tools serialize via lock keys
- **SSE durability**: events persisted to `memento_agent_event`, replayable via `after_seq`/`Last-Event-ID`
- **Multi-provider**: Gemini Interactions API, OpenAI-compatible (Chat Completions + Responses API for reasoning), `fake` for tests
- **Human-in-the-loop**: `propose_backfill` tool sets run to `waiting_for_user`, polls `memento_agent_decision`, 90s default timeout

Tool catalog (36 tools): `fts_search`, `vector_search`, `get_message`, `get_message_batch`, `summarize_thread`, `get_bundle_index`, `get_project_bundle`, `write_section`, `get_concept_bundle`, `cluster_messages_by_subject`, `write_concept_section`, `list_person_messages`, `fts_search_scoped`, `write_facet`, `write_person_attribute`, `record_no_person_attributes`, `write_person_section`, `get_person_network`, `get_group`, `get_cluster`, `find_bridges_between`, `find_missing_collaborators`, `get_person_summary`, `get_project_summary`, `get_concept_summary`, `search_persons`, `search_projects`, `search_concepts`, `detect_gaps`, `detect_gaps_with_results`, `add_project_messages`, `add_concept_messages`, `context_status`, `propose_bundle`, `create_project_draft`, `create_concept_draft`, `propose_backfill`

## Context Management Strategy

Monotonic context accumulation is the biggest structural problem. Key mitigations:
- `get_bundle_index` (body-free index) required as first step before `get_*_bundle`
- `get_message_batch` with `body_char_limit` (default 1200, max 4000)
- `summarize_thread` for deterministic digests instead of full thread fetch
- `context_status` tool reports `budget_level` (normal/watch/low/critical) with actionable guidance
- `MEMENTO_AGENT_CONTEXT_LIMIT_TOKENS` (default 128000) as denominator
- Gemini uses server-side `interaction_id` state; OpenAI-compatible replays transcript each step

Open problems: no trimming/summarization of completed steps mid-run, no dashboard chat pruning, per-agent step limits still global (not differentiated), no search result detail modes yet.

## Person Resolution Algorithm

Pure deterministic, two-stage pipeline in `backend/internal/person`:
1. **Resolve**: normalize emails (lowercase, plus-tag stripping), normalize names (whitespace collapse, forwarder parenthetical stripping), merge passes (plus-tag, exact name, forwarder unwrap, optional Jaro-Winkler + Jaccard fuzzy at 0.92/0.60 thresholds)
2. **Classify**: rule-based exclusion (no-reply, newsletter domains, broadcast patterns, generic roles), then score as `candidate`, `weak_signal`, `candidate_inbound_only`, or `excluded` using 10-message minimum + 0.10 bidirectional score threshold

Person persistence preserves stable IDs via majority overlap and lowest-ID tiebreak. Locked manual links survive resolver reruns.

## Newsletter Detection

Groups messages by sender email, excludes account-owned and human candidates, then classifies via: known newsletter domains, newsletter-like local parts, newsletter-like display names, unsubscribe links in body, recurring sender threshold (default 20 messages). Persisted into `memento_newsletter_source` with upsert/detect-remove snapshot semantics.

## Key Techniques

1. **Deterministic-first pipeline**: canonical person resolution, newsletter detection, and bundle assembly run before any LLM call — the LLM generates prose from pre-assembled evidence, never rediscovers the archive shape from scratch
2. **Outcome tracking with repair**: `outcomeTracker` counts successful tool calls against `OutcomeRequirement` specs; missing required writes trigger one bounded repair turn with a specific "call only the missing tools" prompt; this prevents the "finished but wrote nothing" failure mode
3. **Materialized rollups**: index pages read `memento_*_report` tables rebuilt in transactions; no N+1 archive joins on request path
4. **msgvault as read-only substrate**: hybrid search (default), FTS fallback, vector search for semantic recall; `msgvault serve` HTTP API preferred for concurrent reads (8 parallel tools vs 4 direct SQLite)
5. **SSE durability**: browser disconnect doesn't cancel runs; events replay from `GET /api/internal/agent-runs/{id}/events`
6. **Multi-provider abstraction**: `Provider` interface with `Stream(ctx, ModelRequest, emit func(ModelEvent))` — Gemini Interactions API, OpenAI-compatible Chat Completions, and fake for tests share one contract
7. **Protected writes**: user-edited sections survive regeneration; `write_section` returns `skipped=true` for user-edited sections, and skipped writes don't satisfy completion requirements
8. **Person bootstrap context**: before person enrichment, deterministic bootstrap loads compact person summary, authoritative notes, alias/social summaries, recent compact messages, existing facets/attributes/narrative, generation mode, and cleanup cutoff — all injected into the initial transcript
9. **Draft workflow**: collector agent searches, bundles, and proposes backfill; user reviews before commit; committed project/concept becomes the deterministic boundary

## Design Decisions (from DECISIONS.md)

- June 13: Single-binary distribution (Go serves static UI, no Node.js at runtime)
- June 12: Reasoning-model controls are provider-gated (Responses API for OpenAI, Chat Completions for DeepSeek V4)
- June 4: Demo mode isolated from real archive (separate `data/memento-demo.db`)
- June 4: Running backend DB handles not hot-swapped
- June 4: Demo setup must not require external model or embedding services
- June 11: Ask Sessions are product artifacts, not debug runs (persisted in `memento_ask_session`/`memento_ask_turn`/`memento_ask_context_ref`)
- June 11: Session promotion enters draft workflow (doesn't directly create durable project/concept)

## Code Metrics

- Backend: ~20,000 lines Go (agent runner + server + tools + dimension logic)
- Frontend: ~15,000 lines TypeScript/TSX
- Docs: ~6,000 lines of design documentation
- Tests: comprehensive integration tests for agent runs (1013 lines), providers (629 lines), person merge (455 lines), candidates (236 lines)
- E2E: 7 Playwright specs covering home, navigation, people, projects, concepts, newsletters, ask-memento

## Comparison Notes

Unlike **Mnemo** (Rust sidecar, SQLite + petgraph, no embeddings), Memento embeds AI agents directly in the same process and uses msgvault's existing search infrastructure rather than building its own retrieval.

Unlike **Rowboat** (Obsidian vault, knowledge graph from email+docs+meetings), Memento treats msgvault data as read-only and writes source-attributed living documents — generated prose traces back to specific message IDs rather than floating in a graph.

Unlike **Personal Agents** frameworks (Hermes, clawdBot, Pi), Memento is not a general-purpose agent platform — it's a purpose-built memory layer over a specific archive format (msgvault). The agents are hardcoded per dimension, not user-configurable.

Unlike general RAG systems, Memento uses **deterministic extraction before LLM generation** — the LLM writes prose from pre-assembled evidence bundles, not from search-at-prompt-time retrieval. This is closer to a "curated evidence → narrative compile" pipeline than a question-answering system.
