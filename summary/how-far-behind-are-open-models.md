---
url: https://www.lesswrong.com/posts/rJcCrXyEsJKmmDpWG/how-far-behind-are-open-models
title: "How far behind are open models?"
author: Håvard Tveit Ihle
date_fetched: 2026-06-05
date_published: 2026-05-28
source: LessWrong
tags: [ai, benchmarks, open-source, models, evaluation]
---

# How far behind are open models?

**Author:** Håvard Tveit Ihle
**Published:** 28th May 2026
**Source:** LessWrong (Frontpage, tagged with AI Evaluations, AI Timelines, AI)

## Summary

The author investigates the capability gap between open-weight AI models (downloadable weights) and closed models (API-only), using data from 17 benchmarks (~110 datapoints). The analysis distinguishes between **private benchmarks** (non-public data) and **public benchmarks**.

### Key Results

On **private benchmarks**, open models trail the closed frontier by roughly **8–10 months**. On **public benchmarks**, the gap is roughly **4–6 months**. The gap was "smallest around the time of DeepSeek R1, in Jan 2025, and since then the gap has been growing."

The author notes these are backward-looking figures: today's best open models perform at the level closed models achieved 8–10 months ago.

### Provider Degradation Caveat

Researchers running private benchmarks on Chinese open models may use third-party providers to protect data privacy. However, "third-party providers can have subtly degraded performance when serving open models." If present, this biases the gap larger, especially for private benchmarks.

### Real-World Tasks

The author speculates the gap on real-world tasks is likely even larger than private benchmarks suggest. Open-model developers may inadvertently train toward benchmark-type tasks, while "well-resourced closed labs probably have more access to varied data, more enterprise customers" and less focus on benchmark scores.

### Methodology

Threshold scores are defined per benchmark (typically 5% intervals). For each threshold, the author measures how many months earlier a closed model first crossed it compared to when an open model did. Judgments about whether plausible first-crossers were missing were made by Claude Opus 4.7, with manual overrides where the author felt Opus was too conservative.

The backward-looking perspective defines the gap by associating it with the open model's release date. This ensures "our estimate of the current gap is not biased by the exclusion of currently-open gaps."

### Additional Analyses

- **By category:** Reasoning benchmarks show a larger gap, but all three reasoning benchmarks in the dataset are private, so that's likely the dominant factor.
- **Chinese models only:** Results are basically the same back to Llama 3.1 (July 2024); before that the gap is notably larger.

### Acknowledgements

Data comes primarily from the **Epoch AI Benchmarking Hub**. Claude Opus 4.7 wrote essentially all code and conducted benchmark research, directed by the author.

## Appendix A: Additional Figures

Includes per-benchmark delay timelines for METR Time Horizons (task-completion time in minutes), GPQA Diamond (graduate-level science multiple-choice), MMLU (near-saturated, self-reported), and WeirdML (novel ML-coding tasks).

## Appendix B: Benchmark Score Provenance

The author audited all 17 benchmarks for trustworthiness:

**✅ Independently and comparably run:**
GPQA Diamond, MATH Level 5, OTIS Mock AIME (Epoch-run), plus WeirdML, SimpleBench, and METR (each run end-to-end by a single party).

**❌ Self-reported or submission-based:**
GSM8K (~70% vendor tech-report numbers), MMLU (developer self-reported), MMLU-Pro (community submissions), Terminal-Bench (PR-submitted scaffolds vary), and HLE's open side.

**⚠️ Notable contamination risks:**
- **FrontierMath** — OpenAI funded it and ran its own o3 numbers separately, which can overstate the gap by inflating the closed side.
- **ARC-AGI / ARC-AGI-2** — "semi-private" sets exposed to commercial APIs; closed models receive inputs via first-party APIs, open models via third-party hosts, risking inflated closed scores.

The author concludes: "the risk is over- not under-statement of the gap."

## Comments

**Random Developer** notes that absolute capability thresholds matter for cost-sensitive users. An open model like Sonnet 4.5 level is viable "in the hands of an experienced developer," and there is value in models that "force tight supervision while still automating a lot of boring work."

**Alexander Barry**: "I think this is a sensible methodological approach and puts some nice data behind a common intuition many people have."
