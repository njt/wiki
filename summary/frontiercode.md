---
title: "FrontierCode — Introducing FrontierCode"
url: https://cognition.ai/blog/frontier-code
author: "Eric Lu, Ben Pan, Deniz Birlikci, Sam Lee, Ray Wang, Rohan Choudhury, Fermi Ma, TC Qin, Carlo Baronio, Silas Alberti, and more (Cognition)"
date_fetched: 2026-06-12
date_published: 2026-06-08
tags: [coding-benchmark, eval, agents, code-quality, mergeability]
topics:
  - misc
---

# Introducing FrontierCode | Cognition

Cognition (creators of Devin) introduces FrontierCode, a coding benchmark that asks whether models can write *good* code — not just functionally correct code. Built with 20+ maintainers of 36 flagship open-source repos, each task represents 40+ hours of maintainer work. The benchmark moves beyond pass/fail correctness testing into mergeability: would a human maintainer actually accept this PR?

## Core Thesis

"Correctness is now table stakes." The new question: "can models actually write *good* code?"

## What Makes It Different

1. **Mergeability measurement**: First benchmark to measure code mergeability across six axes: behavioral correctness, regression safety, mechanical cleanliness, test correctness, scope discipline, and code quality.
2. **Open-source maintainer authorship**: 20+ world-class developers built tasks from repos they maintain. They define what "mergeable" means in their repo.
3. **Rigorous QC**: Adversarial testing, calibration, multi-stage review. 81% lower false positive rate compared to SWE-Bench Pro.

## Benchmark Structure

- **Extended**: 150 tasks (full set)
- **Main**: 100 hardest tasks (includes Diamond)
- **Diamond**: 50 hardest tasks

## Metrics

- **Pass rate**: Passes if all "blocker criteria" cleared — criteria a maintainer would consider hard stops.
- **Score**: Weighted aggregate of rubric items. Solutions failing blocking criteria score 0.
- Each model runs 5 times per reasoning effort. Best reasoning level reported.

## Results (Diamond)

- Claude Opus 4.8: 13.4% (best)
- GPT-5.5: 6.3%
- Gemini 3.1 Pro: 4.7%
- Kimi K2.6 (best open-source): 3.8%

GPT 5.5 uses up to 4x fewer tokens than Opus 4.8, achieving better cost-intelligence tradeoff.

## Grading Methods

| Category | Method | How It Works |
|---|---|---|
| Behavioral correctness | classical | Injects test files, runs them |
| Mechanical cleanliness | command | Runs shell command, checks exit code |
| Test correctness | reverse-classical | Agent's tests must fail on original code |
| Complex correctness | adaptive classical (mutagent) | LLM adapts tests to implementation |
| Scope | scope | File boundaries, diff size, semantic locality |
| Code quality | prompt | LLM reviews diff against rubric |

## Criterion Types

- **Blockers**: Must-pass mergeability requirements (correctness, performance, scope)
- **Non-blockers**: Quality signals (style, type safety, readability)

## Novel Grading Methods

**Reverse-Classical**: Agent-written tests must FAIL when run on the original broken codebase. Proves the agent understood the problem enough to write effective tests.

**Code Scope**: Three constraint types — `files` (allowed/denied/modified), `size` (line limits), `semantic` (LLM-based locality checks).

**Adaptive Classical Grading (mutagent)**: LLM surgically patches the test environment to align with the agent's implementation, enabling deterministic tests on open-ended solutions.

## Example Task (jsonschema, C++)

Create `LOG_WARNING()` in `src/logger.h`. Warnings to stderr, always print (ignore --verbose), auto-prefix "warning:". Replace all `warning: <message>` instances. Opus 4.8 failed because it mixed `LOG_WARNING()` with raw `std::cerr` calls — behaviorally identical but idiomatically wrong (bakes in assumption that both use same stream). Reviewed by Andrew He (ecnerwala), #2 US Codeforces competitor, 2x IOI gold medalist, Cognition founding engineer.

## QC Pipeline (5 stages)

1. **Design**: Prefer classical tests for correctness, prompt-based for soft qualities
2. **Hack Report**: Author tries to cheat the rubric (false positives) and writes alternative valid solution (false negatives). Devin also attempts to hack the rubric.
3. **Calibration**: Author writes 4 solutions targeting 0-100% scores
4. **Review**: Pod lead → Cognition researcher → multi-round iteration. Researchers solve tasks themselves for a random subset.
5. **Re-Review**: Most tasks cycle through multiple iterations

## External Quotes

**Tomer Nosrati (Celery, 28.6k stars)**: "Where others grade like a CI, FrontierCode grades like a tech lead."

**Martin McKeaveney (Budibase, 28k stars)**: "Each task is calibrated to a depth that simply hasn't been seen before in LLM benchmarking."

**Merlijn Vos (uppy, 30.8k stars)**: Called it "a milestone for AI models respecting subjective quality in the real world."

**Claudio Costa (Mattermost, 37k stars)**: "The human experience encoded in its evals: years of judgment about what makes code high-quality and worthy of merging."

## Key Data Points

- 20+ maintainers, 36 flagship repos, 40+ hours per task
- 150 tasks (Extended), 100 (Main), 50 (Diamond)
- Prompts one-third the length of SWE-Bench Pro's
- 81% lower false positive rate than SWE-Bench Pro
- Top Diamond score: 13.4% — benchmark is far from saturated
- Language diversity: triple SWE-Bench Pro's represented languages
