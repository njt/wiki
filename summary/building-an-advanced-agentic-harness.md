---
url: https://data4sci.com/blog/building-an-advanced-agentic-harness
title: Building an Advanced Agentic Harness
site: Data for Science (data4sci.com)
date_fetched: 2026-08-06
topics:
  - agent-orchestration
  - agent-memory-and-context
---

A technical deep-dive that upgrades a basic ~35-line agent loop into a production-shaped harness through seven composable primitives. Each primitive is motivated by a specific, predictable failure mode of naive agents, and each is built as a small, testable component rather than hidden behind a framework.

**Typed tools** replace hand-validated arguments with Pydantic models that generate JSON Schema for the LLM, validate at runtime, and fail fast before expensive tool calls. **Plan-as-DAG** replaces sequential step-by-step execution with a dependency graph; the Planner emits the full graph up front, the executor runs ready nodes concurrently with a semaphore cap, and validation catches hallucinated dependencies before execution. **Tiered memory** replaces context-window dumping with working/episodic/semantic tiers, retrieving top-k by embedding similarity under a hard character budget with explicit (not silent) truncation. **Verification hierarchy** runs cheap deterministic checks first (e.g., "are all requested cities in the report?"), escalating to an expensive LLM judge only on survivors — the generator is never grading its own homework.

**Planner/Worker/Critic role separation** splits a single confused prompt into three narrow agents with single contracts, each independently testable and swappable. **Multi-dimensional budgeting** tracks tokens, tool calls, wall time, and estimated dollars simultaneously; a single `pressure()` scalar drives graceful degradation (skip the LLM judge above 0.9, halt at 1.0). **A structured tracer** captures per-step identity, semantics, economics, and verdicts as flat JSON events, enabling latency-by-role plots, budget-pressure-over-time charts, and token-by-role breakdowns.

The running example is a city comparison agent — deliberately simple but chosen so that nine independent lookups feed one aggregation, naturally decomposing into parallel work. All tools use mock data for reproducibility; the LLM backend is abstracted behind a pluggable `LLMProvider` interface with a deterministic `MockProvider` for separating orchestration bugs from model-quality issues. The orchestrator is deliberately thin: it assembles context, gets a plan, executes with budget-charging callbacks, verifies with pressure-aware degradation, and stores outcomes in episodic memory — with an informed re-planning loop for missing-information errors instead of blind retries.

The article explicitly leaves evals, retrieval benchmarks, and specialized worker pools for a future post. Its thesis is that composability — not framework magic — is the difference between a proof-of-concept and an extensible harness.
