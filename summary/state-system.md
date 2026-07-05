---
url: https://github.com/dbmcco/state-system
title: State System — A Model-Mediated Organizational State Layer
author: David McCowan (dbmcco)
date_fetched: 2026-06-05
date_published: 2025-2026
---

# State System — Full Repo Analysis

## Project Overview

State System is an **organizational state layer** defined as a Python product repo (MIT-licensed, ~11.4K LoC Python, ~40 JSON schemas, ~50 examples, ~45 tests). It is not a deployed application — it's a **definition surface**: schemas, contracts, a CLI runtime, source-module contracts, package rendering, freshness audits, federation packs, and conformance tests. The README calls it "a generic model-mediated substrate for tracking organizational state."

The project has two separable forms:
1. The **product repo** (this repo) — defines schemas, contracts, runtime code, migrations, documentation
2. A **deployed state system instance** — holds actual runtime state, read models, freshness evidence, database configs

## Architecture

### Core Pipeline: Source Event → Trigger → Review → Commit → Recent → Package

The fundamental loop is stored as `trace-manifest.json` files run by `trace_runner.py`:

1. **Source Event** (ingested from external systems like GitHub, Linear, CRM) — validated against `source-event.schema.json`, deduplicated via 4 identity mechanisms (idempotency key, source_event_id, semantic fingerprint, field transition), watermarked for ordering
2. **Trigger** — derived from source event (`runner.py:_trigger_from_source_event()`) — wraps the event with actor, summary, evidence refs, candidate state refs
3. **Review Packet** — built by `ReviewPacketBuilder` — bundles trigger + resolved evidence + state snapshots + journal entries + persona + governance constraints + allowed outputs (8 types: no_op, state_proposal, memory_proposal, promotion_proposal, action_proposal, rollup_request, missing_evidence, review_signal)
4. **Model Output** — produced by an LLM (external to this repo) — contains state_proposals, memory_proposals, action_proposals, promotion_proposals, rollup_requests, and a review_signal
5. **Commit** (`committer.py:Committer.commit()`) — validates model output against schemas, checks for pending approvals, rejects proposals with unresolved evidence or protected field patches, materializes state snapshots via journal application, persists commit result and review signal
6. **Recent Change Index** (`recent_changes.py`) — indexes the change with persona routing (which personas should see this), freshness metadata (stale_after, watermark_refs), and opportunity class hints
7. **Context Package** (`context_packages.py`) — assembles recent changes + state snapshots + journal entries + memory + evidence + governance into a persona-scoped package for agent consumption

### Storage Layer

`stores.py` defines `JsonFileStore` — a flat-file JSON store organized by collection directories (`state/<collection>/<id>.json`). The `StateStoreBundle` dataclass creates 19 named stores (state_objects, source_events, review_packets, journals, memory, rollups, review_signals, commits, recent_changes, context_packages, agent_activations, agent_responses, instance_capabilities, instance_agent_packages, instance_preflight_results, instance_source_freshness, company_capabilities, company_preflight_results, source_freshness).

`replay()` reads all records sorted by `created_at` timestamp, providing append-only journal semantics. `create()` enforces unique IDs with `RecordExistsError`.

### CLI Surface

`cli.py` (~1,719 lines) is a monolithic argparse-based CLI exposing 35 subcommands organized into 19 collections. The CLI is designed as a **deterministic read surface** — every command produces JSON to stdout. Key commands include:

- `validate` — validate all examples against schemas
- `trigger` — ingest a source event
- `review` — build a review packet from a source event
- `commit` — commit model output (state proposals → journals + materialized snapshots)
- `build-package` — build a context package for an agent
- `capture-response` — capture agent response text as evidence
- `north-star-answer` — build the North Star substrate from packages
- `trace-run` — run a trace manifest (deterministic replay of the full pipeline)
- `operational-loop-run` — run the operational loop (wraps trace-run + operator summary)
- `instance-scaffold` — scaffold a new state instance
- `fleet-refresh-run` — validate source freshness across a fleet of instances

### Schema System

The project has ~40 JSON schemas in `schemas/`. Key schemas:

- `state-object.schema.json` — the core entity: 18 types (project, deal, client, relationship, campaign, meeting, obligation, person, organization, mission, strategy, principle, role, onboarding, norm, decision_area, capability, agent, operating_picture), 8 state families (organizational_identity, operating, work, relationship, knowledge, role_and_persona, onboarding, governance), 4 state traits (slow_changing, dynamic, developmental, rollup), situations with 8 states (watching → resolved), actions with 5 statuses
- `state-instance.schema.json` — 7 instance kinds (company, personal, project, portfolio, household, research, other), 5 sensitivity levels, entity references, federation
- `model-review-packet.schema.json` — the bridge between source events and model: trigger + evidence packet + state/journal/memory/persona/governance context + 8 allowed outputs
- `model-proposal-output.schema.json` — what the model returns: state_proposals, memory_proposals, action_proposals, promotion_proposals, rollup_requests, review_signal
- `commit-result.schema.json` — what the committer produces: status, accepted refs, pending approvals, rejected proposals, review signal
- `instance-agent-package.schema.json` — the agent-facing package: source readiness, question routes, federation packs, tool action refs, answer contracts, fallback policies, gap behaviors

### Instance Model

Instances are the deployment unit. An instance has a kind (company/personal/project/etc.), a primary entity, sensitivity default, governance refs, and federation relationships. Instances are scaffolded via `instance-scaffold` CLI command and managed through capability packs, preflight checks, and source freshness audits.

The instance hierarchy:
- **Instance** — a deployable state root (e.g., `state_instance.sampleco`)
- **Instance Capability Pack** — declares connectors, tools, agents, and runtime constraints
- **Instance Preflight** — validates connector/tool/agent readiness before use
- **Instance Source Freshness** — tracks source data freshness with watermarks
- **Instance Agent Package** — the rendered agent-facing artifact combining all of the above

### Agent Integration

`agent_consumers.py` is the agent-facing surface. It:
1. Renders packages as structured text for agent consumption
2. Captures agent responses as evidence records
3. Enforces the "captured agent output is evidence, not truth" boundary

The rendering distinguishes between `context_package` (simpler, for human/persona review) and `instance_agent_package` (richer, with source readiness, question routes, federation packs, tool actions, answer contracts, fallback policies, gap behaviors).

### Federation Model

Cross-instance queries are governed by `InstanceFederationPack` — a boundary object declaring that one local instance can query a remote instance through named routes. Federation packs enforce:
- Identity boundaries (subjects/entities remain owned by remote source)
- Materialization policy (default: `local_materialization=false`)
- Freshness policy (checked time, watermark, stale-after)
- Subject-note policy (subject notes can demote/explain but not filter silently)
- Output policy (safe summaries with evidence refs, no raw remote corpora)

### Source Module Extension

Source modules (`source-modules.md`) are the open-source extension point. Each module declares connector type, allowed instance kinds, access modes, preflight/freshness/index/tool contracts, module modes (live API, historical cache, local sync, export, federated query), read/write/correction surfaces, output policy, gap behavior, and governance defaults.

The conformance suite (`test_open_source_ecosystem_conformance.py`) checks that capability-pack connector types have source modules, tool actions reference known modules, and question-route tools have tool action contracts.

## Key Techniques

1. **Deterministic replay**: Trace manifests (`trace-manifest.schema.json`) define every step — seed_records, source_event, review, commit, recent_change, context_package, render_package, agent_activation, capture_response — as paths to JSON fixtures. `trace_runner.py` runs them deterministically. This means the entire pipeline from source event to agent response capture is replayable and testable.

2. **Model/code boundary**: The model owns interpretation (what changed, what matters, what's uncertain, what actions make sense). Code owns integrity (schemas, evidence references, permissions, persistence, auditability, replay). This is a clean and principled separation — the model proposes, code enforces invariants.

3. **Idempotent source event ingestion**: `SourceEventIngestor` deduplicates incoming events by 4 identity mechanisms — idempotency key, source_event_id, semantic fingerprint, and field transition identity (a computed key from `source_system:object_ref:field:old_value:new_value`). Duplicates are recorded with a `duplicate_of_ref` and `duplicate_reason` rather than being dropped silently.

4. **Watermark ordering**: `_watermark_status()` compares incoming event watermarks against prior watermarks from the same source system, flagging events as `"out_of_order"` if they arrive late. This is the temporal integrity mechanism for distributed source events.

5. **Materialization with protected fields**: `materialize_snapshot()` merges a journal entry's state_patch into a state object snapshot, but `PROTECTED_PATCH_FIELDS` prevents the model from modifying certain fields (id, type, primary_family, etc.). If a model proposal touches protected fields, the committer rejects it.

6. **Persona routing**: Recent changes carry `candidate_persona_routes` — each route specifies a persona_ref, included flag, relevance_tier, and routing_reason. Packages filter changes by persona. Personalities aren't prompt styles but interpretive lenses with specific responsibilities, watched domains, and authority boundaries.

7. **Package pressure questions**: Not tests in the traditional sense — they're real operational questions with structural assertions against package JSON. They check that packages expose the right routes, source coverage, tool/action refs, answer policies, freshness gaps, and federation boundaries before an agent would answer.

8. **North Star answer**: `north_star_answer.py` builds a deterministic JSON substrate answering 8 canonical questions (current state, why, what changed, evidence, uncertainty, responsibility, next actions, broader effects) from package data alone. It deliberately does NOT ingest raw source data, own synthesis, or authorize execution — preserving the model/code boundary.

## Design Decisions

1. **Flat file store over database**: `JsonFileStore` is aggressively simple — one JSON file per record, directories as collections, replay by timestamp sort. This is the right call for a product definition surface. Deployed instances should swap this for Postgres/vector DB as needed. The schema contracts remain valid regardless of storage backend.

2. **Monolithic CLI over microservices**: 35 subcommands in a single ~1,719-line `cli.py`. Deliberate choice — this is a definition and tooling repo, not a production service. CLI commands are composable via Unix pipes (JSON in, JSON out). If this were a production system, each concern would be its own service.

3. **JSON as the universal format**: Everything is JSON — schemas, examples, runtime state, CLI output. This is a strong choice for an agent substrate because JSON is machine-parseable, LLM-friendly, and schema-validatable. The trade-off is human readability — JSON in Obsidian isn't great, but this repo isn't meant for direct human consumption.

4. **Evidence-first architecture**: Every claim (journal entry, memory entry, state change) carries `evidence_refs`. The committer rejects proposals with unresolved evidence. Freshness is tracked per-source with watermarks and stale-after timestamps. You cannot commit state changes without evidence — this is the core integrity guarantee.

5. **Federation over centralization**: State instances federate — cross-instance queries go through declared routes with explicit boundaries. No raw data sync. No hidden cross-instance materialization. This is designed for multi-instance deployments (personal + company + portfolio) where governance boundaries matter.

6. **Code owns integrity, model owns interpretation**: The project's most important design principle. Code validates schemas, enforces evidence requirements, prevents protected field mutation, deduplicates events, and tracks watermarks. The model (LLM) decides what changed, what matters, and what actions make sense. Neither layer crosses into the other's territory.

7. **Personas are first-class, not style prompts**: A persona in State System is a structured interpretive lens — responsibilities, facets, watched domains, authority boundaries, anti-patterns. Laura notices different things than an operations or finance agent. This is deeper than most "persona" implementations which are just system prompt variations.

## Innovation Points

1. **The model/code boundary as architectural principle**: Most agent systems either put everything in the model (vibe-driven) or everything in code (deterministic workflows). State System draws a bright line: model interprets, code enforces. Both are necessary, neither substitutes for the other.

2. **Append-only journal with materialized snapshots**: State changes are committed as journal entries (append-only, evidence-carrying) and then materialized into current snapshots. You get both the audit trail and the current view. The materialization function is deterministic and reversible — replay all journals to reconstruct any snapshot from any point in time.

3. **Trace manifests as deterministic test fixtures**: The `trace-run` command runs a JSON manifest that defines every pipeline step. This means you can test the entire system — from source event ingestion to agent response capture — without an actual LLM. Test fixtures are replayable, diffable, and serve as documentation.

4. **Source freshness as a first-class concern**: Most agent systems assume data is fresh. State System tracks per-source freshness with watermarks (Git commit SHAs, API timestamps), stale-after cutoffs, and explicit `requires_refresh_before_external_action` flags. Agents are instructed to treat stale sources as visible caveats, not silent failures.

5. **Deterministic North Star substrate**: The `north-star-answer` command produces a JSON answer to "What is the current state?" without calling any model. It synthesizes package data, freshness audits, source readiness, and evidence refs into a structured substrate that preserves uncertainty and gaps. The model can then use this substrate to write prose — but the grounding is deterministic and inspectable.

## Line Count & Scale

- Total Python LoC: ~11,433 (31 source files)
- Largest module: `cli.py` (1,719 lines — CLI surface)
- Second largest: `instance_agent_packages.py` (1,436 lines — agent package construction)
- Core pipeline modules: `agent_consumers.py` (578 lines), `instance_understanding_surface.py` (552 lines), `reporting.py` (522 lines), `trace_runner.py` (423 lines), `contracts.py` (416 lines), `committer.py` (377 lines), `north_star_answer.py` (368 lines), `runner.py` (307 lines)
- Schemas: 40 JSON schema files
- Examples: ~50 JSON fixture files
- Tests: ~45 test files
- Dependencies: Python ≥3.11, hatchling build system, zero runtime dependencies (stdlib only)

## Critical Analysis

**What's brilliant**: The model/code boundary is the right abstraction for agent systems. The trace manifest system gives you deterministic replay of the entire pipeline — this is production-grade observability in a product definition repo. The evidence-first architecture (every claim carries evidence refs, committer rejects unresolved evidence) is the strongest integrity guarantee I've seen in an agent substrate.

**What's missing**: There's no actual vector search or embedding pipeline — the "interpreted index" (`interpreted_index.py`, 254 lines) reads from the file store but doesn't implement semantic retrieval. The README says semantic retrieval "must remain evidence plumbing, not hidden organizational judgment," but the plumbing itself isn't built. Similarly, there's no LLM integration code — the model is always external, called by some other system, with its output passed to the `commit` CLI command. This is correct for the product definition boundary but means anyone deploying this needs to build their own model integration.

**What's risky**: The flat file store works at definition scale but will be a bottleneck at deployment scale — replaying 10,000 journal entries by reading 10,000 JSON files and sorting by timestamp is O(n log n) I/O. The monolithic CLI has 35 subcommands and no plugin system — adding a new integration means editing the ~1,719-line `cli.py`. These are acceptable trade-offs for a v0.1.0 product definition, but they should be replaced before production deployment.

**What's genuinely novel**: Personas as interpretive lenses (not prompt styles), the `PROTECTED_PATCH_FIELDS` mechanism (code-enforced constraints on model proposals), the North Star substrate (deterministic answer generation that preserves gaps and uncertainty), and the "captured agent output is evidence, not truth" boundary. I haven't seen these patterns combined in any other agent substrate.

## Comparison

Compared to [[Agent Memory and Context]] tools like Mnemo (knowledge graph from conversations) or Rowboat (knowledge graph from email/docs): State System operates at a higher level — it's organizational memory, not personal memory. It tracks deals, projects, obligations, campaigns, and institutional context, not conversation history or document summaries. It's a CRM + project tracker + operations dashboard + institutional memory, unified by a consistent evidence model.

Compared to [[Guardrails and Feedback Loops]]: State System's guardrails are structural (schemas, evidence requirements, protected fields) rather than behavioral (policies, evaluation prompts). The committer will reject a malformed proposal regardless of its content — this is compile-time safety for agent output.

Compared to [[Agent Orchestration]]: State System doesn't orchestrate agents. It provides the state substrate that agents read from and write to. It's the canonical "what is true right now" that agents consult before acting. The orchestration layer sits on top of State System, not inside it.

Compared to [[Elysia]]: Elysia constrains tool choice per decision-tree node. State System constrains state mutations per schema and evidence. Different constraints for different problems — Elysia for tool execution safety, State System for state integrity.
