---
url: https://firethering.com/granite-4-1-ibm-open-source-model-family/
title: "Granite 4.1: IBM's 8B Model Competing With Models Four Times Its Size"
author: Mohit Geryani
date_fetched: 2026-05-15
date_published: 2026-04-30
source: Firethering
---

# Granite 4.1: IBM's 8B Model Competing With Models Four Times Its Size

**Author:** Mohit Geryani
**Date:** April 30, 2026
**Source:** Firethering

---

## Overview

IBM released Granite 4.1, a family of open-source language models aimed at enterprise use. Three sizes are available under Apache 2.0, trained on 15 trillion tokens with a notably meticulous pipeline.

The standout result: the 8B model uses a dense architecture—no mixture-of-experts tricks, no extended reasoning chains—yet matches or beats Granite 4.0-H-Small, a 32B MoE model with 9B active parameters, across nearly every benchmark. The author notes this is "either very impressive or it means the old model was underbuilt. Probably both."

## How It Was Built

Granite 4.1 comes in 3B, 8B, and 30B configurations, all using the same decoder-only dense transformer design and identical training pipeline. "The only difference between them is size." No MoE routing, sparse layers, or reasoning chains that inflate token counts.

IBM ran five distinct training phases, each with different data mixtures and learning rate schedules. Phase 1 started broad (59% CommonCrawl, 20% code, 7% math). By Phase 2, math jumped to 35% and code to 30%. Phases 3 and 4 blended chain-of-thought reasoning trajectories and instruction data with the highest-quality web content. Phase 5 extended the context window to 512K tokens for the 8B and 30B.

## Data Quality Pipeline

Before fine-tuning, IBM built an LLM-as-Judge filtering system that evaluated every assistant response across six dimensions: instruction following, correctness, completeness, conciseness, naturalness, and calibration. Responses below threshold were cut. Some things triggered automatic rejection regardless of score—hallucinations, false premises, incorrect computations.

In RAG settings, responses not grounded in retrieved documents counted as hallucinations. In tool-calling scenarios, outputs were checked against allowed tools and parameter schemas. A separate rule-based pipeline handled structure, length, formatting, and deduplication. The final output was 4.1 million samples—"a deliberately curated 4.1 million."

## Four-Stage Reinforcement Learning

The RL process is where the author finds the most instructive details, particularly IBM's honesty about something breaking mid-training.

**Stage 1:** Joint training across nine domains simultaneously—math, science, logical reasoning, instruction following, structured output, text-to-SQL, temporal reasoning, general chat, and in-context learning. Joint training prevents the model from forgetting earlier domains as it improves on later ones.

**Stage 2:** RLHF training on general chat prompts using a reward model. AlpacaEval scores jumped roughly 18.9 points on average. But this caused math benchmark scores to drop—both GSM8K and DeepMind-Math regressed.

**Stage 3:** A brief identity and knowledge calibration run of about 40 training steps to stabilize self-representation and knowledge boundaries.

**Stage 4:** A dedicated math RL run to recover what RLHF had damaged. It worked: GSM8K surpassed the fine-tuned baseline by about 3.8 points on average, and DeepMind-Math recovered by roughly 23.5 points on average.

## Benchmark Results

The author presents self-reported benchmarks from IBM's own evaluation harness, with the caveat that methodology always deserves scrutiny. Key scores:

| Benchmark | What It Tests | 3B | 8B | 30B |
|---|---|---|---|---|
| IFEval | Instruction following | 82.1 | 87.1 | 89.7 |
| BFCL V3 | Tool calling | 60.8 | 68.3 | 73.7 |
| GSM8K | Math reasoning | 87.0 | 92.5 | 94.2 |
| DeepMind-Math | Advanced math | 64.6 | 80.1 | 81.9 |
| EvalPlus | Coding | 67.1 | 80.2 | 82.7 |
| ArenaHard | Real-world chat quality | 37.8 | 69.0 | 71.0 |
| MMLU-Pro | General knowledge | 49.8 | 56.0 | 64.1 |

The 30B tops IBM's own BFCL V3 tool calling chart at 73.7, ahead of Gemma-4-31B at 72.7. The 8B at 68.3 surpasses the prior Granite 4.0-H-Small at 64.7, and the 3B at 60.8 clears Qwen3-8B at 60.2—a model twice its size.

On IFEval, Gemma leads at 94.1, but the 8B at 87.1 is essentially tied with Qwen3.5-9B at 87.2. The 30B at 89.7 beats every Qwen model on the chart regardless of size. On math, the 8B hits 92.5 on GSM8K and 80.1 on DeepMind-Math. On coding, EvalPlus puts the 8B at 80.2 and the 30B at 82.7.

The 3B is described as "the quiet story here"—82.1 on IFEval, 87.0 on GSM8K, 60.8 on BFCL V3—strong numbers for edge deployment or cost-constrained inference.

## 512K Context Window

IBM used a staged extension approach. Rather than jumping directly to 512K, they went 32K first, then 128K, then 512K. Each stage used the same data mix as Phase 4 until the final extension, where they switched to 80% books and 20% code repository data. After each stage, IBM performed a model merge, combining the long-context checkpoint with earlier weights to preserve short-context behaviors.

RULER benchmark scores for the 8B base: 83.6 at 32K, 79.1 at 64K, and 73.0 at 128K. The 30B holds up better: 85.2, 84.6, and 76.7. "There's degradation as context grows, which is expected and honest, but the scores don't fall off a cliff." The 3B only extends to 128K.

## How to Run It

The quickest path is via Ollama. The 3B runs on most consumer machines; the 8B needs more headroom; the 30B requires a GPU machine. All three are on Hugging Face under ibm-granite. vLLM and Transformers support them out of the box, and IBM offers API access. FP8 quantized variants are available at roughly half the memory footprint. Apache 2.0 licensing means commercial use is clean.

## Who Should Care

The author identifies the 8B as the sweet spot for anyone needing reliable tool calling, predictable latency, and a clean license. The 3B is worth considering for edge use cases or tight inference budgets. The 30B is for when the ceiling is needed and hardware is available.

The closing assessment: "What IBM built here is a production-first model family from a team that clearly spent more time fixing problems than announcing them." The four-stage RL pipeline that caught and corrected a mid-training regression is the kind of detail "that doesn't make headlines but absolutely shows up in real-world reliability."
