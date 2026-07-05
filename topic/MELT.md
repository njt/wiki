# MELT

Open-source benchmark harness for evaluating long-lived memory in AI agent systems. Tests whether memory survives correction, contradiction, decay, and consolidation — not just retrieval accuracy. Built by Shisa AI alongside their shisad memory system when they found no existing eval that tested how memory works *over time*.

---

## Architecture

MELT is a **harness, not a dataset**. It runs replayable suites of cases against any memory system through a pluggable SUT adapter contract:

```
Config → Runner → SUT Adapter → System Under Test
                ↓
         Suite / Fixture
                ↓
      Scoring + Methodology
                ↓
         JSON Report
```

**Core pipeline** (`src/melt/`, ~3,500 LOC, zero PyPI deps, Python 3.11+ stdlib only):

- **`runner.py`** (2,708 lines): Main engine. Loads fixtures, creates SUT adapters, iterates cases, executes step sequences (8 op types), runs multi-granularity scoring, generates harness answers, performs judged QA, builds reproducibility envelopes. Supports multi-run aggregation, checkpoint resume, per-case SUT restart, partial-run error recovery.
- **`config.py`** (720 lines): TOML config with `${ENV_VAR}` expansion, CLI override merging, exhaustive key validation, SHA-256 config hashing for reproducibility.
- **`schema.py`** (328 lines): Dataclass schema for suites, cases, steps, and expectations with full validation. 8 step ops, 5 expectation kinds.
- **`scoring.py`** (399 lines): Deterministic scorers — retrieval (R@k, nDCG@k at turn/session/document granularity), lifecycle assertions (8 distinct predicates), memory state scoring (export vs black-box modes), token F1, keyword match.
- **`lifecycle.py`** (1,874 lines): MELT-native lifecycle suite. 13 behavioral axes × generated variants → 7-373 cases. Each axis has a dedicated behavioral template exercising the specific memory dynamic it claims to measure.
- **`sut/fake.py`** (377 lines): In-process deterministic oracle with full memory semantics — decay scoring, timestamp-gated visibility, predicate contradiction detection, supersession tracking. Passes everything by construction, validating that generated cases are satisfiable.
- **`adapters/`**: Import adapters for LongMemEval (500 Q, S/M variants), LoCoMo (10 conversations, 1986 Q, Category 5 audit), RHELM (10 scenarios, 1305 Q). Plus dependency-free RHELM downloader using HTTP range requests.

**SUT adapter contract** (`sut/base.py`): 12-operation Protocol over JSONL-over-stdio (shisad) or HTTP (memobase) — `hello/hello_ack`, `metadata`, `reset`, `ingest`, `memory_write`, `consolidate`, `manage`, `tick`, `memory_export`, `query`, `answer`, `shutdown`. Versioned contract (currently B2) with capability negotiation.

## Key Techniques

**Lifecycle assertion taxonomy.** Eight assertions beyond "did it retrieve?" Each probes a specific temporal behavior:
- `stored_fact_matches` — basic capture quality
- `superseded_fact_absent` — correction replaced old data
- `contradiction_detected` — conflicts flagged, not merged
- `historical_fact_retained` — as-of query returns past state
- `decayed_below_threshold` — stale info aged below threshold
- `core_memory_stable` — durable facts survive full consolidation
- `negative_evidence_absent` — distractors excluded from results
- `scope_isolated` — out-of-scope facts don't leak
- `abstention_on_unknown` — system declines when it should

Each assertion maps to one of the 13 lifecycle axes, giving per-axis pass rates that diagnose *which* temporal behaviors a memory system gets wrong.

**Dual write surface testing.** Scripted track uses explicit `memory_write` operations (keys, values, predicates, supersession chain). Agentic track uses raw `ingest` events followed by autonomous `manage` calls. Same axis tested through different code paths — a system might handle structured facts but fail at extracting them from conversation.

**Temporal/as-of queries.** Queries carry `as_of_time` parameters. A correction case writes old fact at T1, superseding fact at T2, queries at T3 with `as_of_time` = T1.5 — the old value must return while the new value must be absent from current queries. This is impossible to test with static retrieval benchmarks.

**Fake oracle as satisfiability validator.** The deterministic fake SUT implements full memory semantics and passes every behavioral case by construction. If the fake can't pass a case, the case is broken. Published results show fake at 1.000000 across all axes, shisad at 0.953571, Memobase at 0.446429 on the full lifecycle suite.

**Memory state dual introspection.** Export mode scores against a full SUT memory dump with token efficiency metrics. Black-box mode scores from probe query evidence. Without `memory_export` support, write-precision collapses to recall and dedupe rate is trivially 1.0 — the report makes this transparency explicit. Only the fake SUT currently supports export.

**Conversation-level LoCoMo cases.** Ingests each conversation once, asks all questions against the same state — vs the legacy one-question-per-case shape that re-ingests the same conversation 155-199 times. Reduces SUT churn by 10-20× for full runs.

**Config-as-reproducibility-hash.** Resolved config (defaults + file + CLI + env) is SHA-256 hashed. Two reports with the same config hash used identical settings. Every report records fixture hash, SUT identity, contract version, runner commit, seed, run count, top-k setting, preliminary/final status, and judge identity.

## Design Decisions

| Decision | Why |
|---|---|
| **Zero PyPI deps** | No install friction, no supply chain risk. Cost: reimplements TOML, HTTP, SQLite from stdlib. Python 3.11+ only (needs tomllib). |
| **Config-first** | Every setting in TOML with exhaustive validation. Enables reproducible runs. The 720-line config validator is the cost of confidence. |
| **Preliminary vs final status** | Single run or dev split = preliminary; held-out with 5+ runs = final. Prevents overclaiming from single-run results. |
| **Built-in fixtures** | Smoke/mini/full/stress tiers built in. Run lifecycle evals with zero downloads. Cost: generated cases can't match real conversation diversity. |
| **Top-k bypass warnings** | When top_k >= session count, report warns retrieval scores are for sanity, not competition. Honest but could be gamed. |
| **Baselines as first-class** | Raw_verbatim, no_context, gold_evidence, full_transcript baselines run alongside SUT. Enables leakage detection. Cost: up to 4× answer calls for judged QA. |
| **Per-case checkpoints** | Every case writes checkpoint after completion. Resume picks up where it left off. Essential for multi-hour LoCoMo runs (published: 379s for Memobase lifecycle full). |
| **Separate answer/judge models** | Answer can be cheap (GPT-5.4 Mini, $0.75/M input), judge can be careful (GPT-5.4, $2.50/M input). Published LoCoMo: $4.46 for shisad, $8.92 for Memobase. |
| **Deterministic scorers default** | Retrieval and token-F1 judging are free. LLM judge is opt-in. Makes CI/pre-commit eval runs cost zero. |

## Comparison Notes

MELT is the only benchmark that systematically tests **temporal memory dynamics** — correction, contradiction, as-of recall, decay, and maintenance — rather than static retrieval accuracy.

- **vs LongMemEval** (Wu et al. 2024): Retrieval from 62-487 fixed sessions. MELT adapts it but adds lifecycle assertions the original doesn't have.
- **vs LoCoMo** (Maharana et al. 2024): QA over 10 long conversations. MELT adds Category 5 audit, conversation-level cases, and the full methodology envelope.
- **vs RHELM**: Structured document retrieval. MELT imports it with per-document/section/turn/question-type/source-type granularity.
- **vs MemBench** (Tan et al. ACL 2025): Retrieval-only with fixed knowledge base. No temporal dynamics.
- **vs FrontierCode**: Coding benchmark, different domain. Shares the "harness over dataset" philosophy.
- **vs general eval methodology** ([[Demystifying Evals for AI Agents]]): MELT instantiates split tracking, preliminary/final, top-k transparency, and baseline checks as concrete implementation.

Related memory infrastructure that could be evaluated through MELT's SUT adapter contract: [[Sawtooth Memory]], [[Mnemo]], [[LocalAI]] (includes memory service), [[Claude-Mem]].

---

*Tags: #tool #benchmark #ai-memory #evaluation*

*Sources: [[summary/melt]]*
*Last updated: 2026-06-15*
