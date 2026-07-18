---
url: https://blog.shisa.ai/posts/melt/
title: "MELT: Testing Long-Lived Memory for Agentic AI"
author: Leonard Lin
date_fetched: 2026-07-18
date_published: 2026-07-13
---

The piece opens by describing an awkwardly basic problem the team encountered while building the memory system for ShisaD: determining whether an AI agent's memory is actually working.

The author notes that most popular memory benchmarks use a common recipe — loading a fixed conversation transcript, asking questions about it, and scoring answers. This mostly tests "retrieval and reading comprehension over a frozen snapshot."

Real memory, Lin argues, gets harder after the snapshot changes. A system should know a user's new employer, stop presenting the old one as current, and still recall the previous one. It should preserve durable health info like an allergy through consolidation while letting a one-off doctor's appointment fade. It should also keep conflicting claims separate and say "I don't know" when evidence is lacking.

Existing benchmarks generally don't exercise that full loop, so the team built **MELT (Memory Evaluation for Lifecycle Testing)**, described as an open benchmark runner for testing memory over time.

The TL;DR summary states that MELT tests whether an agent memory system can "write, update, maintain, and retrieve the right information as its state changes." The current release includes a native lifecycle benchmark with 13 behavioral axes and 217 cases in the `full` fixture, scripted and agentic memory tracks, adapters for LongMemEval, LoCoMo, and RHELM under a common report format, runnable system adapters for ShisaD and Memobase plus a deterministic reference adapter, and reproducibility metadata with checkpointing and resume support.

## A Memory Benchmark Should Test Memory

Static-conversation benchmarks answer whether a system can find enough evidence in a long history to answer correctly. They don't necessarily reveal whether the system stored the right things, handled a correction, preserved history, survived maintenance, or forgot appropriately. The author observes that surprisingly simple baselines — raw embeddings, grep, or dumping the full transcript into a sufficiently long context window — can sometimes match or beat much more elaborate memory architectures.

Lin says this doesn't make retrieval benchmarks bad. MELT runs LongMemEval and LoCoMo because they remain useful comparison points.

MELT also supports RHELM, a Microsoft-developed benchmark for evolving user profiles, heterogeneous sources, and questions requiring temporal synthesis. RHELM is closer in spirit to MELT's goals, though it remains primarily QA over a constructed history. MELT's native lifecycle suite additionally tests "explicit writing, correction, supersession, consolidation, decay, and forgetting."

The article includes a table of behavioral areas the native suite focuses on:

- **Write quality** — whether the system captured the useful fact rather than surrounding noise
- **Structured semantics** — preservation of source IDs, timestamps, and supersession metadata
- **Correction** — whether a newer fact becomes current without erasing the older fact's history
- **Contradiction** — whether conflicting claims remain distinguishable instead of blending into one false memory
- **Temporal recall** — answering both "what is true now?" and "what was true then?"
- **Maintenance** — whether consolidation and decay preserved durable facts while letting stale ones recede
- **Core memory** — whether identity, constraints, and long-lived preferences survived
- **Multi-hop recall** — bringing together related evidence from separate sessions
- **Abstention** — declining to answer when memory doesn't support an answer
- **Retrieval hygiene** — excluding high-similarity distractors and superseded evidence
- **Scope and source** — keeping users/workspaces isolated and handling chat, email, documents, and summaries
- **Operations** — whether retries, time advancement, and maintenance cycles leave memory in a valid state

(The implementation reports these as 13 separate axes; the table groups several for readability.)

## Scripted Memory vs. Agentic Memory

MELT separates two distinct questions. In the **scripted lifecycle track**, the harness explicitly tells the system what to write, supersede, consolidate, or decay. This tests execution: whether a memory system performs a precise lifecycle operation correctly.

In the **agentic memory track**, MELT sends raw timestamped events and lets the system decide what should become memory. This tests policy: whether the system decides to retain a durable preference, ignore noise, update stale information, preserve a real conflict, or forget ephemera on its own.

Lin emphasizes these are different claims. A system can have excellent storage semantics but poor extraction policy, or make good write decisions atop a weak retrieval layer. MELT keeps track, axis, write surface, time horizon, query mode, retrieval granularity, evidence type, and lifecycle operation separate in the report rather than "compressing everything into one suspiciously tidy number."

The article also discusses an observability problem. Some systems can export internal memory units; others are black boxes. MELT supports both but labels them differently. Export-capable systems get direct write/update/forget policy scoring. Black-box systems are judged through retrieval and behavior, which the author notes is "inherently weaker."

## Benchmark and Harness

MELT is both a native benchmark and a system-agnostic runner. A suite supplies sessions, operations, probes, and expected outcomes; a System Under Test (SUT) adapter translates MELT's versioned operations into whatever interface the memory system exposes.

The same runner can execute MELT lifecycle cases or normalize established benchmarks into a common report envelope. Today it ships with:

- `lifecycle`: `smoke` (7 cases), `mini` (48), `full` (217), and `stress` (373)
- `longmemeval`: built-in smoke plus user-supplied full datasets
- `locomo`: smoke and full runs, with Category 5 audit warnings and conversation-level ingestion
- `rhelm`: smoke and full runs, plus a dependency-free dataset downloader

The adapter contract covers reset, ingest, structured writes, consolidation, time-aware query, answer generation, and shutdown. The built-in ShisaD adapter uses JSONL over a subprocess; the Memobase adapter talks to its HTTP service. A new system can use either pattern without changing benchmark scoring code.

## Early Results: One "Memory Score" Is Not Enough

The author states that current public results are still preliminary: development fixtures, single runs, and known caveats. They are presented as useful diagnostics, not leaderboard claims.

On MELT's 217-case `lifecycle-v4` full fixture, the runs show:

| System | Lifecycle pass | Retrieval R@3 | Correction | Contradiction | Temporal | Maintenance |
|---|---|---|---|---|---|---|
| ShisaD | 95.4% | 90.3% | 100.0% | 100.0% | 23.5% | 100.0% |
| Memobase | 44.6% | 13.4% | 56.7% | 13.3% | 23.5% | 13.3% |

Lin notes surprise at ShisaD's lifecycle score, explaining that although MELT grew out of work on ShisaD, the benchmark cases were developed independently with no evaluation-specific tuning. ShisaD's memory system was "explicitly designed around agentic memory lifecycle mechanics, so this is a test it should be structurally well suited to."

The key insight is not that one aggregate number is higher. The per-axis view immediately reveals that temporal/as-of recall is a weakness for both systems in this run — exactly the kind of failure a static aggregate hides.

Comparing the same systems on full LoCoMo judged question answering, using GPT-5.4 Mini as answer model and GPT-5.4 as judge:

| System | Questions | Judged QA score |
|---|---|---|
| Memobase | 1,986 | 42.0% |
| ShisaD | 1,986 | 6.2% |

The ordering flips completely. Lin clarifies this is not a contradiction and certainly not evidence that either table is a universal ranking. The lifecycle suite tests controlled memory behaviors; LoCoMo tests whether retrieved context supports QA over long conversations. ShisaD's retrieval context was effective on lifecycle cases but weak for LoCoMo QA. Memobase's was the reverse. Different tests exposed different systems.

The article lists important caveats: every result is a single development run marked preliminary; the lifecycle runs trigger a retrieval-bypass warning because `top_k=3` matches the number of canonical sessions in built-in cases; neither real adapter exposes full memory export, so write-policy metrics use weaker black-box proxies; and LoCoMo has known dataset and judging problems.

Lin closes by observing that "evals are hard, and agentic memory evals are still nascent." Memory spans many behaviors, making any single score easy to overread. MELT is the team's attempt to add nuance and tease apart those dimensions.

The article directs interested readers to the Agentic Memory Research repository and invites contributions via GitHub.
