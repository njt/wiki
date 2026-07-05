---
url: https://github.com/shisa-ai/MELT
date_fetched: 2026-07-05
backfilled: true
---

MELT is an open benchmark runner for evaluating long-lived memory in AI agent systems. It tests whether a memory system remains useful and correct as facts change, old information expires, new evidence arrives, and the system performs maintenance over time.

Most memory benchmarks evaluate retrieval from a fixed transcript or preloaded conversation history. MELT is built for the harder lifecycle questions: did the system store the right thing, update it when the world changed, keep durable facts through consolidation, avoid overclaiming, and retrieve the right evidence at the right time?

MELT is two things at once: a **harness** for running memory evaluations under a
reproducible report envelope, and a **native lifecycle benchmark** that probes
those harder questions directly. The same harness also runs adapted versions of
**LongMemEval**, **LoCoMo**, and **RHELM**, so you can compare a system on
established retrieval benchmarks and on MELT's lifecycle tasks side by side.

MELT is not tied to one memory implementation. Any system can be evaluated by implementing the System Under Test (SUT) adapter contract.

- **Researchers**who need reproducible lifecycle tasks and report envelopes for comparing memory architectures.
- **Memory-system developers**who want targeted tests for write quality, correction, contradiction handling, consolidation, decay, recall, and abstention.
- **Teams evaluating agent memory**who need evidence beyond "can it retrieve a fact from a transcript?"

| Dimension | Question MELT asks | 
|---|---|
| Write quality | Did the system capture useful information from the session? | 
| Correction | Did newer facts replace superseded older facts? | 
| Contradiction | Did conflicting claims remain distinguishable instead of being merged into a false fact? | 
| Temporal recall | Can the system answer as-of a point in time and keep historical facts available after they change? | 
| Maintenance | Did consolidation and decay preserve useful memories while letting stale ones expire? | 
| Core memory | Did durable preferences and identity facts survive over time? | 
| Multi-hop recall | Can the system retrieve related evidence from separate sessions? | 
| Abstention | Does the system decline when it should not know the answer? | 
| Methodology | Are splits, seeds, top-k settings, baselines, judge settings, and provenance recorded? | 

- A Python CLI runner: `melt run`
- The MELT-native lifecycle suite, with scripted and agentic tracks, covering write quality, correction, contradiction, temporal/as-of recall, consolidation and decay, core-memory stability, multi-hop recall, and abstention
- Import adapters for **LongMemEval**,**LoCoMo**, and**RHELM**that normalize those benchmarks into the same report envelope, plus a dependency-free RHELM downloader (`melt download-rhelm`)
- A JSONL-over-stdio SUT adapter contract, with built-in adapters for a
deterministic in-process `fake`SUT, the`shisad`subprocess SUT, and the`memobase`HTTP service
- Optional harness answer generation and judged QA (deterministic local scoring or OpenAI-compatible providers), with no-context, gold-evidence, and full-transcript answer baselines
- Per-case checkpointing with resume, and JSON reports carrying provenance,
scoring metadata, warnings, and `preliminary`/`final`result status

MELT is an early release. The runner and current suites are usable; larger native corpora and additional adapters are expected to grow over time. Reference runs across SUTs are published in RESULTS.md.

```
git clone https://github.com/shisa-ai/MELT.git
cd MELT
uv sync
# Run the minimal smoke fixture against the deterministic fake SUT.
uv run melt run --sut fake --suite smoke --fixture smoke --output-dir results
# Run the MELT-native lifecycle smoke suite.
uv run melt run --sut fake --suite lifecycle --fixture smoke --top-k 3 --output-dir results
# Exercise harness-generated answers and deterministic judged QA.
uv run melt run --sut fake --suite smoke --fixture smoke --answer-mode harness --judge --output-dir results
# Add answer-control baselines for leakage and upper-bound checks.
uv run melt run --sut fake --suite smoke --fixture smoke --answer-mode harness --judge \
  --baseline --no-context-baseline --gold-evidence-baseline --full-transcript-baseline \
  --output-dir results
# Resume a long run from completed per-case checkpoints.
uv run melt run --sut fake --suite smoke --fixture smoke --answer-mode harness --judge \
  --resume-from-checkpoint-dir results/checkpoints/<invocation-id> \
  --output-dir results
# Download RHELM without installing Hugging Face datasets or hub packages.
uv run melt download-rhelm --output-dir data/rhelm
# Run the test suite.
uv run pytest
```
Each benchmark run writes a JSON report under the configured output directory. The report records the suite, fixture hash, SUT identity, contract version, runner commit, seed, run count, top-k setting, baseline status, warnings, and whether the result is preliminary or final. Answer-generation runs also record the claim type, answer mode, answer prompt hash, judge prompt hash, and compact usage metadata.

Long answer runs write per-case checkpoints under
`<output-dir>/checkpoints/<invocation-id>/<run-id>/`. `--resume-from-checkpoint-dir`
can point at either that run-specific directory or its invocation parent; MELT
reuses completed cases and continues with the remaining ones.

Harness-answer runs can use deterministic local scoring or an OpenAI-compatible HTTP provider configured in TOML:

```
[answer]
mode = "harness"
[answer.model]
provider = "openai_compatible"
model = "answer-model"
base_url = "${ANSWER_MODEL_BASE_URL}"
api_key = "${ANSWER_MODEL_API_KEY}"
max_output_tokens = 4096
[judge]
enabled = true
mode = "llm"
[judge.model]
provider = "openai_compatible"
model = "judge-model"
base_url = "${JUDGE_MODEL_BASE_URL}"
api_key = "${JUDGE_MODEL_API_KEY}"
max_output_tokens = 4096
```
Official-like profile shortcuts pin prompt IDs and record deviations in the report envelope:

`uv run melt run --suite locomo --fixture smoke --answer-profile locomo_official_like --judge`| Suite | Fixtures | Purpose | 
|---|---|---|
| `smoke` | `smoke` | Minimal end-to-end check for runner, SUT contract, scoring, and report generation. | 
| `lifecycle` | `smoke`,`mini`,`full`,`stress` | MELT-native lifecycle benchmark (scripted + agentic tracks): write quality, correction, contradiction, temporal/as-of recall, consolidation and decay, core memory, multi-hop recall, and abstention. | 
| `longmemeval` | `smoke`,`external`,`full` | LongMemEval records normalized into MELT's report envelope. | 
| `locomo` | `smoke`,`external`,`full` | LoCoMo records normalized into MELT's report envelope, including Category 5 audit metadata. | 
| `rhelm` | `smoke`,`external`,`full` | RHELM records normalized into scenario-level cases with document, section, turn, question-type, and source-type reporting. | 

Full LongMemEval and LoCoMo runs require locally supplied dataset files. RHELM can be downloaded with MELT's dependency-free downloader:

```
uv run melt download-rhelm --output-dir data/rhelm
uv run melt run --sut fake --suite rhelm --fixture external --dataset-path data/rhelm --output-dir results
```
MELT does not vendor restricted external datasets. LoCoMo external/full fixtures
use conversation-level cases by default, so each conversation is ingested once
and all of its questions are asked against that state. Use
`--case-granularity qa` only when you explicitly need the legacy one-question-
per-case shape.

Retrieval-only MELT runs do not call an answer model or LLM judge. API cost is only incurred when you enable a model-backed answer path, a model-backed judge, or answer baselines. Every report records answer/judge calls, tokens, cache usage, elapsed time, and max RSS so you can price a run against your provider's current rates.

Typical suite sizes:

| Suite / fixture | Local snapshot size | Cases / questions | Retrieval queries | Model calls when answer + judge are enabled | 
|---|---|---|---|---|
| `smoke`/`smoke` | built in | 1 case, 1 QA expectation | 1 | 1 answer + 1 judge | 
| `lifecycle`/`smoke` | built in | 7 cases | 7 | 0; lifecycle has no QA expectations | 
| `lifecycle`/`mini` | built in | 48 cases | 48 | 0; lifecycle has no QA expectations | 
| `lifecycle`/`full` | built in | 217 cases | 217 | 0; lifecycle has no QA expectations | 
| `lifecycle`/`stress` | built in | 373 cases | 373 | 0; lifecycle has no QA expectations | 
| `longmemeval`/`smoke` | built in | 2 cases, 1 QA expectation | 2 | 1 answer + 1 judge | 
| `longmemeval`/`full`variant S | user supplied; 265 MB in the local reference snapshot | 500 questions | 500 | up to 500 answers + 500 judges | 
| `longmemeval`/`full`variant M | user supplied; 2.6 GB in the local reference snapshot | 500 questions | 500 | up to 500 answers + 500 judges | 
| `locomo`/`smoke` | built in | 1 case, 1 QA expectation | 1 | 1 answer + 1 judge | 
| `locomo`/`full`, Category 5 excluded | user supplied; 2.7 MB in the local reference snapshot | 10 conversation cases, 1540 questions | 1540 | up to 1540 answers + 1540 judges | 
| `locomo`/`full`, Category 5 included | user supplied; 2.7 MB in the local reference snapshot | 10 conversation cases, 1986 questions | 1986 | up to 1986 answers + 1986 judges | 
| `rhelm`/`smoke` | built in | 1 scenario, 2 QA expectations | 2 | 2 answers + 2 judges | 
| `rhelm`/`full` | user supplied; 53 MB in the local reference snapshot | 10 scenarios, 1305 questions | 1305 | up to 1305 answers + 1305 judges | 

Baselines multiply answer calls. For example, running SUT answers plus
`--no-context-baseline`, `--gold-evidence-baseline`, and
`--full-transcript-baseline` can add three extra answer calls per question before
judging. Context size also depends on the SUT: a system that returns longer
retrieval snippets can cost more to answer and judge than one with the same
question count.

As a concrete reference, the published LoCoMo Category 5 included runs in
RESULTS.md used 1986 answer calls and 1986 judge calls per SUT
with a GPT-5.4 Mini answer model and GPT-5.4 judge. At the prices recorded in
that results section, equivalent cost was $4.46 for shisad and $8.92 for
Memobase because the returned contexts differed. The published lifecycle `full`
runs used zero MELT answer/judge calls and took about 95 seconds for shisad and
379 seconds for the successful Memobase run on the local reference machine.

```
melt run -> Config -> Runner -> SUT Adapter -> System Under Test
                         |
                         v
                  Suite / Fixture
                         |
                         v
              Scoring + Methodology
                         |
                         v
                  JSON Report
```
- A suite fixture defines replay sessions, memory operations, probes, and expected outcomes.
- The runner sends those steps to a SUT adapter.
- The memory system ingests events, performs maintenance, retrieves evidence, or answers probes.
- MELT scores the run and writes a report with reproducibility metadata.

To evaluate a memory system with MELT, implement a SUT adapter. The adapter can wrap an in-process library, a subprocess, or a service, but it must expose the versioned SUT operations expected by the runner:

- `hello`/- `hello_ack`
- `metadata`
- `reset`
- `ingest`
- `memory_write`
- `consolidate`
- `query`
- `answer`
- `shutdown`

The adapter guide describes the JSONL-over-stdio protocol, capability negotiation, message shapes, and expected error behavior: docs/adapters.md.

MELT reports are designed to make evaluation claims easier to audit:

- Dev, validation, and held-out splits are recorded separately.
- Single-run reports are marked `preliminary`.
- Top-k settings are recorded, and bypass-like retrieval settings produce warnings.
- Raw-verbatim baselines can be run beside architecture results.
- Retrieval-only and answer-generation claims are labeled separately.
- Judge identity and prompt hashes are included when judge scoring is enabled.
- LoCoMo Category 5 inclusion and dataset-audit caveats are explicit.
- LoCoMo conversation-level runs weight metrics by question, not by conversation.
- Report loading validates fixture identity and key provenance fields.

```
uv sync
uv run pytest
uv run melt --help
uv run melt run --help
```
Project structure:

```
src/melt/             Runner, scoring, config, reports, adapters, and SUT APIs
src/melt/suites/      MELT-native lifecycle suites
src/melt/adapters/    Standard benchmark import adapters
src/melt/sut/         SUT adapter interfaces and registry
docs/adapters.md        SUT adapter authoring guide
docs/PLAN-lifecycle.md  MELT-native lifecycle benchmark design
docs/PLAN-answer.md     Answer-generation and judged-QA design plan
docs/INITIAL.md         Original benchmark design draft
RESULTS.md              Published reference runs and methodology notes
tests/                  Unit and behavioral tests
```
MELT was developed during the course of ShisaD's memory system development when we discovered there was no memory eval that properly tested how memory systems work over time.

- Agentic Memory Benchmark Survey
- LongMemEval: Wu et al. 2024 (arxiv 2410.10813)
- LoCoMo: Maharana et al. 2024 (arxiv 2402.17753)
- MemBench: Tan et al. ACL 2025 (arxiv 2506.21605)

Copyright 2026 Shisa AI

Licensed under the Apache License, Version 2.0. See LICENSE for details.
