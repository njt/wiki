---
url: https://github.com/shisa-ai/MELT
title: MELT - Memory Evaluation for Lifecycle Testing
author: Shisa AI
date_fetched: 2026-06-15
date_published: 2026-05-22
topics:
  - agent-memory-and-context
---

# MELT — Full Repo Analysis

MELT (Memory Evaluation for Lifecycle Testing) is an open-source benchmark harness for evaluating long-lived memory in AI agent systems. It tests whether a memory system remains useful and correct as facts change, old information expires, new evidence arrives, and the system performs maintenance over time. It is both a harness for running memory evaluations under a reproducible report envelope and a native lifecycle benchmark that probes lifecycle questions directly. The same harness runs adapted versions of LongMemEval, LoCoMo, and RHELM.

Version: 0.2.0 (Alpha). License: Apache 2.0. Python 3.11+ with zero PyPI dependencies (stdlib only). Built alongside shisad's memory system development.

## Architecture

MELT follows a harness-based benchmark runner pattern:

```
melt run → Config → Runner → SUT Adapter → System Under Test
                        ↓
                 Suite / Fixture
                        ↓
              Scoring + Methodology
                        ↓
                 JSON Report
```

**Core modules (src/melt/, ~3,500 LOC):**

- `cli.py` (310 lines): argparse-based CLI with `melt run` and `melt download-rhelm` commands. Full config override support via `--override KEY=VALUE` and 30+ individual flags.
- `config.py` (720 lines): TOML config loading with env var expansion (`${VAR}`), CLI override merging, exhaustive validation of every config key, and SHA-256 config hashing for reproducibility.
- `runner.py` (2,708 lines): The main execution engine. Loads fixtures, creates SUT adapters, iterates cases, executes steps (ingest/memory_write/consolidate/query/answer/manage/tick/memory_export), runs scoring, generates answer attempts, performs judged QA, and builds report envelopes. Supports multi-run aggregation, checkpoint resume, per-case SUT restart, and partial-run error recovery.
- `schema.py` (328 lines): Dataclass-based schema for SuiteManifest, SuiteFixture, EvalCase, CaseStep, and Expectation with comprehensive validation. Expectation kinds: retrieval, qa, abstention, lifecycle, memory_state. Allowed ops: 8 step types.
- `scoring.py` (399 lines): Deterministic scorers — retrieval (R@k, nDCG@k at turn/session/document granularity), keyword match, token F1, lifecycle assertion pass (8 distinct assertions), memory state scoring (export mode with token efficiency vs black-box mode with probe-based observation).
- `contracts.py` (34 lines): Shared constants — MELT schema version "1", report schema version "4", B2 SUT capability sets.
- `answering.py`: Harness answer generation via deterministic or OpenAI-compatible providers, with context packet building from ranked/chronological/grouped evidence.
- `judge.py`: LLM judge with deterministic fallback, retry logic, strict/permissive failure modes.
- `llm.py`: HTTP LLM client for OpenAI-compatible APIs.
- `llm_cache.py`: SQLite-backed deterministic cache for answer and judge LLM calls with namespace isolation.
- `methodology.py`: Evaluation envelope building, metric summary, top-k bypass detection, report status determination.
- `reports.py`: JSON report writing with validation.
- `redaction.py`: Secret/credential redaction from configs and error messages before serialization.
- `fixtures.py`: Built-in smoke fixture loading.
- `baselines/verbatim.py`: RawVerbatimBaseline — in-process baseline SUT that echoes ingested events.

**SUT adapters (src/melt/sut/):**
- `base.py`: SutAdapter Protocol defining 12 operations (metadata, reset, ingest, memory_write, consolidate, manage, tick, memory_export, query, answer, shutdown)
- `registry.py`: Adapter registry with 6 entries (fake, shisad, memobase — available; memv — blocked; mira, karta — followup). Factory pattern with validation.
- `fake.py` (377 lines): In-process deterministic oracle SUT with full memory semantics — decay scoring, timestamp-gated visibility, predicate-based contradiction detection, supersession tracking, memory export with unit alignment. Passes all lifecycle cases by construction, serving as validation that generated cases are satisfiable.
- `shisad.py`: JSONL-over-stdio subprocess adapter for shisad memory SUT. Handles capability negotiation, hello handshake, stdin/stdout message framing, timeout/resource limits.
- `memobase.py`: HTTP service adapter for Memobase project APIs. Maps Memobase event-gist text back to MELT source IDs for evidence attribution.

**Suite (src/melt/suites/):**
- `lifecycle.py` (1,874 lines): MELT-native lifecycle benchmark. 13 behavioral axes × generated variants = 217 cases (full) to 373 cases (stress). Each axis has a dedicated behavioral template: correction (supersession), contradiction (predicate conflict + consolidation), temporal (as-of query before supersession), maintenance (decay threshold), core_memory (full consolidation survival), multi_hop (cross-session aggregation), abstention (unanswerable query), negative_retrieval (distractor exclusion), privacy_scope (out-of-scope leak prevention), operational (consolidation + time tick survival). Dual tracks: scripted (explicit memory_write) and agentic (ingest → manage → probe). Two introspection modes: export (full memory dump) and black_box (probe-based).

**Import adapters (src/melt/adapters/):**
- `longmemeval.py`: LongMemEval (Wu et al. 2024) import adapter. 500 questions, supports S (62 sessions) and M (487 sessions) variants.
- `locomo.py`: LoCoMo (Maharana et al. 2024) import adapter. 10 conversations, 1986 questions (1540 with Category 5 excluded). Supports conversation-level and QA-level case granularity. Includes dataset error ceiling tracking, audit catalog, and Category 5 opt-in with warning.
- `rhelm.py`: RHELM import adapter. 10 scenarios, 1305 questions. Includes dependency-free RHELM downloader (HTTP range requests against Hugging Face). Reports by document, section, turn, question type, and source type.

## Key Techniques

1. **Lifecycle assertion taxonomy**: 8 distinct assertions beyond simple retrieval — `stored_fact_matches`, `superseded_fact_absent`, `contradiction_detected`, `historical_fact_retained`, `decayed_below_threshold`, `core_memory_stable`, `negative_evidence_absent`, `scope_isolated`, `abstention_on_unknown`. Each probes a specific memory behavior that retrieval benchmarks can't measure.

2. **Dual write surface architecture**: Scripted track uses explicit `memory_write` operations with structured payloads (keys, values, predicates, supersession). Agentic track uses raw `ingest` events followed by autonomous `manage` calls. Tests the same axis through different code paths — a system might handle explicit facts but fail at extracting them from conversation.

3. **Temporal/as-of query support**: Queries carry `as_of_time` parameters. A correction case writes old fact at T1, superseding fact at T2, then queries at T3 with `as_of_time` between T1 and T2 — the old value must return. This tests whether the system preserves historical state, not just current state.

4. **Fake oracle as satisfiability validator**: The deterministic fake SUT implements full memory semantics (decay scoring with configurable half-life, timestamp-gated visibility, predicate-based contradiction detection, supersession tracking, memory export with source-aligned units). It passes everything by construction, proving the generated cases are satisfiable, not impossible.

5. **Memory state scoring with dual introspection**: Export mode scores against a full SUT memory dump with token efficiency metrics. Black-box mode scores from probe query evidence. Without export, write-precision collapses to recall and dedupe rate is trivially 1.0 — the report makes this transparent.

6. **Per-dimension metric decomposition**: Every case and expectation carries rich tags (lifecycle_axis, write_surface, time_horizon, query_mode, evidence_type, lifecycle_operation, retrieval_granularity). The scoring system automatically generates per-axis metric breakdowns, so you can see that a system fails on contradiction but passes on write_quality.

7. **Conversation-level LoCoMo cases**: Instead of the legacy one-question-per-case shape (which re-ingests the same conversation N times for N questions), conversation-level cases ingest each conversation once and ask all questions against the same SUT state. Reduces SUT churn by 10-20× for full LoCoMo runs.

8. **LLM caching with namespace isolation**: SQLite-backed deterministic cache keyed on (namespace, model, prompt_hash, context_hash). Enables incremental re-runs where only changed prompts hit the API. Particularly valuable for baseline comparisons where most answer calls are identical across runs.

9. **Secret redaction pipeline**: Before any config is serialized (for hashing, reports, or error messages), a redaction pass strips API keys, tokens, passwords, and URL-embedded credentials using URL parsing, query parameter inspection, and fragment analysis. Prevents credential leaks in published reports.

10. **Config-as-reproducibility-hash**: The resolved config (defaults + file + CLI + env) is SHA-256 hashed and included in every report. Two reports with the same config hash used the same settings, making results auditable and comparable.

## Design Decisions

| Decision | Trade-off |
|---|---|
| **Zero PyPI dependencies** | No pip install needed, no supply chain risk. Cost: reimplements TOML parsing, HTTP clients, and SQLite from stdlib. Limits to Python 3.11+ (needs tomllib). |
| **Config-first, not code-first** | Every setting lives in TOML with exhaustive validation. Enables reproducible runs and audit trails. Cost: 720-line config validator is nontrivial maintenance burden. |
| **Preliminary vs final status** | Single-run or dev split = preliminary; held-out with 5+ runs = final. Prevents overclaiming from single-run results. Cost: slightly pedantic for a project in alpha. |
| **Built-in fixtures over external datasets** | Smoke/mini/full/stress tiers built in. You can run lifecycle evals without downloading anything. Cost: generated cases can't match the diversity of real conversation data. |
| **Top-k bypass detection** | When top_k >= session count, report carries warning. Prevents gaming retrieval scores with oversized k. Honest but could be gamed by SUTs that return everything. |
| **Baselines as first-class** | Raw_verbatim, no_context, gold_evidence, full_transcript baselines run alongside SUT. Enables leakage detection and upper-bound estimation. Cost: multiplies answer calls by up to 4× for judged QA runs. |
| **Per-case checkpointing** | Every case writes a checkpoint after completion. Resume picks up where it left off. Essential for multi-hour LoCoMo runs. Cost: checkpoint I/O on every case adds overhead. |
| **Separate answer and judge model configs** | Answer model can be cheap/fast (GPT-5.4 Mini), judge model can be expensive/careful (GPT-5.4). Published runs use this split to keep costs down. |
| **LOCOMO Category 5 as explicit opt-in** | Adversarial category requires `--include-category-5` flag. Report records inclusion and adds audit warning. Recognizes that Category 5 is not standard in published LoCoMo results. |
| **Deterministic scorers as default** | Retrieval scoring and token-F1 judging work without API calls. LLM judge is opt-in. Makes basic eval runs free and fast — important for CI/pre-commit use. |

## Comparison Notes

**vs existing memory benchmarks:**
- LongMemEval evaluates retrieval from a fixed transcript of 62-487 sessions. MELT adapts it but adds lifecycle assertions.
- LoCoMo evaluates QA over 10 long conversations. MELT adds Category 5 audit, conversation-level cases, and the report envelope.
- RHELM evaluates retrieval from structured documents. MELT imports it with granular per-document/section/turn/question-type/source-type metrics.
- MemBench (Tan et al. ACL 2025) is retrieval-only with a fixed knowledge base. MELT probes the full temporal lifecycle.
- EverMemBench — surveyed in research/benchmarks but not adapted into MELT.

**vs other eval harnesses:**
- FrontierCode (Cognition) — mergeability benchmark for coding agents. Different domain but shares the "harness over benchmark" philosophy.
- stupidmeter — lightweight model benchmarking. Different approach (retro UI, leaderboard-focused) vs MELT (reproducibility-focused report envelope).
- Demystifying Evals for AI Agents — general eval methodology. MELT instantiates many of those principles (split tracking, preliminary/final, top-k transparency, baseline checks).

**vs memory infrastructure tools:**
- Sawtooth Memory — 4-tier async memory middleware. MELT could evaluate Sawtooth-backed systems via the SUT adapter contract.
- Mnemo — local-first knowledge graph sidecar. MELT could evaluate it similarly.
- LocalAI — includes a memory service in its gRPC backend. Could be wrapped as a MELT SUT.

**Key differentiation:** MELT is the only benchmark that systematically tests temporal memory dynamics — correction (does a newer fact replace an older one?), contradiction (are conflicting claims flagged, not merged?), as-of recall (can the system answer about past state?), decay (does stale information drop below threshold?), and maintenance (does consolidation preserve durable facts while aging ephemeral ones?). Existing benchmarks test static retrieval accuracy; MELT tests whether memory behaves correctly as the world changes.
