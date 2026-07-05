# State System

An organizational state layer — a Python product repo that defines the schemas, contracts, and runtime for tracking what's true about an organization (deals, projects, people, obligations, campaigns, decisions, agents) with append-only journals, evidence-first commits, and persona-scoped agent packages. It's the missing state substrate between your tools and your agents: the canonical "what is true right now" that agents consult before acting.

---

## Architecture

State System defines a **deterministic pipeline** from external source events to agent-readable packages:

```
Source Event → Trigger → Review Packet → Model Output → Commit → Recent Change → Context Package
```

**`runner.py` (307 lines)** ingests source events with 4-level deduplication (idempotency key, source_event_id, semantic fingerprint, field transition identity) and watermark ordering to detect out-of-order events.

**`committer.py` (377 lines)** is the integrity gate. When an LLM proposes state changes, the committer:
- Validates the output against `model-proposal-output.schema.json`
- Rejects proposals with unresolved evidence refs or protected field patches
- Queues proposals requiring human approval as `pending_approval`
- Materializes accepted proposals as journal entries and updated state snapshots

**`stores.py` (120 lines)** provides the persistence layer — `JsonFileStore`, a flat-file JSON store with one file per record, directories as collections, and timestamp-sorted replay for append-only journal semantics. 19 named collections in the `StateStoreBundle`.

**`cli.py` (1,719 lines)** exposes 35 subcommands as a deterministic read surface — JSON in, JSON out, composable via Unix pipes.

**`trace_runner.py` (423 lines)** runs trace manifests for deterministic replay of the full pipeline from source event to agent response capture, without requiring an actual LLM.

Key schemas (~40 total):
- `state-object.schema.json` — 18 entity types, 8 state families, 4 state traits, situations with 8 lifecycle states
- `state-instance.schema.json` — 7 instance kinds, 5 sensitivity levels, federation
- `model-review-packet.schema.json` — bridge from source events to model with 8 allowed output types
- `model-proposal-output.schema.json` — what the model returns: state_proposals, memory_proposals, action_proposals, promotion_proposals, rollup_requests
- `instance-agent-package.schema.json` — the rendered agent-facing artifact: source readiness, question routes, federation packs, tool actions, answer contracts

## Key Techniques

**Model/code boundary**: The model owns interpretation (what changed, what matters, what's uncertain). Code owns integrity (schemas, evidence, permissions, persistence, replay). The model proposes state changes, but code rejects anything that lacks evidence, touches protected fields, or violates governance constraints. This is the cleanest separation of model judgment from system integrity I've seen in an agent substrate.

**Evidence-first commits**: Every journal entry and memory entry carries `evidence_refs`. The committer (`committer.py:100-156`) rejects any proposal whose evidence refs don't resolve. Freshness is tracked per-source with watermarks and `stale_after` timestamps. You literally cannot commit state changes without evidence — this isn't a policy suggestion, it's enforced by code.

**Idempotent source event ingestion**: `SourceEventIngestor` (`runner.py:40-112`) deduplicates by 4 identity mechanisms. The clever one: `_field_transition_identity()` (`runner.py:275-289`) computes a key from `source_system:object_ref:field:old_value:new_value` — if the same field transition arrives twice, it's a duplicate regardless of event ID.

**Protected field patches**: `materializer.py` defines `PROTECTED_PATCH_FIELDS` — fields the model is never allowed to modify (id, type, primary_family, etc.). The committer rejects any proposal that touches them (`committer.py:127-137`). This is compile-time safety for state mutations.

**Deterministic North Star substrate**: `north_star_answer.py` (368 lines) builds a JSON answer to 8 canonical questions (What is the current state? Why? What changed? What evidence? What's uncertain? Who's responsible? What next? Broader effects?) from package data alone — no model calls, no API queries. Gaps, stale sources, and unresolved evidence are preserved as visible fields rather than smoothed over.

**Personas as interpretive lenses**: Personas in State System (`persona.schema.json`) specify responsibilities, facets, watched domains, authority boundaries, and anti-patterns — not prompt styles. The same state change routed to Laura (marketing) produces different context than routed to Patrick (engineering), because the persona determines what's relevant and what's excluded.

## Design Decisions

**Flat file store** over database: Deliberately simple — one JSON file per record, directories as collections. Right for a product definition surface. Deployed instances should swap for Postgres/vectors. The schema contracts are backend-agnostic.

**JSON everywhere**: Schemas, examples, runtime state, CLI output — all JSON. Machine-parseable, LLM-friendly, schema-validatable. Human-unfriendly, but this is infrastructure, not prose.

**Federation over centralization**: Cross-instance queries go through `InstanceFederationPack` with explicit boundaries — no raw data sync, no hidden materialization. Identity stays with the owning instance. Designed for multi-instance deployments (personal + company + portfolio) where governance boundaries are real.

**Trace manifests as test fixtures**: The `trace-run` command runs a JSON manifest defining every pipeline step. This means you can test the full system — source event → agent response — without an LLM. Fixtures are replayable, diffable, and serve as living documentation.

**Zero runtime dependencies**: Python stdlib only. `pyproject.toml` lists only `hatchling` as a build dependency. The repo is self-contained.

## Comparison Notes

Compared to **[[Agent Memory and Context]]** tools like Mnemo or Rowboat: State System is organizational memory, not personal memory. It tracks deals, projects, obligations, and institutional context, not conversation history. Higher level, unified by a consistent evidence model.

Compared to **[[Guardrails and Feedback Loops]]**: State System's guardrails are structural (schemas, evidence requirements, protected fields) rather than behavioral (evaluation prompts). The committer rejects malformed proposals regardless of their semantic content — this is compile-time safety for agent output.

Compared to **[[Agent Orchestration]]**: State System doesn't orchestrate agents. It provides the state substrate they read from and write to. The orchestration layer sits on top, not inside.

Compared to **[[Elysia]]**: Elysia constrains tool choice per decision-tree node. State System constrains state mutations per schema and evidence. Different surface — Elysia for tool safety, State System for state integrity.

Compared to **[[Smart Models Dumb Pipes]]**: State System is the canonical "dumb pipe" for organizational state — deterministic replay, schema enforcement, evidence tracking. The model is the "smart" layer that interprets meaning and proposes changes.

## What's Missing

No actual LLM integration code — the model is always external. No vector search implementation — the "interpreted index" reads files but has no embedding pipeline. The flat file store won't scale beyond definition scale. The monolithic CLI needs a plugin system for adding integrations without editing 1,719 lines of argparse. All of these are correct for a v0.1.0 product definition but need attention before production deployment.

---

#project #agents #memory #architecture #governance

*Source: [[summary/state-system]]*
*Last updated: 2026-06-05*
