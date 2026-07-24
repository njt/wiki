---
url: https://poolside.ai/blog/introducing-laguna-s-2-1
title: Introducing Laguna S 2.1
author: Poolside team
date_fetched: 2026-07-25
date_published: 2026-07-21
---

# Introducing Laguna S 2.1

Poolside's blog post introducing their latest coding model, released July 21, 2026.

## Model Overview

Laguna S 2.1 is a **Mixture-of-Experts (MoE) model** with **118B total parameters** and **8B activated parameters per token**, supporting a **1M-token context window** in both thinking and no-thinking modes. Training to launch took under nine weeks, starting pre-training on **4,096 NVIDIA H200 GPUs on May 22, 2026**.

## Key Claims

- "The most capable agentic coding model in its weight class by a wide margin."
- Scores **70.2% on Terminal-Bench 2.1** in its agent harness with thinking enabled, ranking 11th on the TB 2.1 leaderboard.
- On **DeepSWE v1.1**, scores **40.4%** (13th place), notably outperforming DeepSeek-V4-Pro-Max (1.6T params) at 9.0%.
- Compact size makes it "uniquely suitable for complex work on local machines" — runs on a single NVIDIA DGX Spark.

## Full Benchmark Scores

| Benchmark | Laguna S 2.1 | Comparison (frontier) |
|---|---|---|
| Terminal-Bench 2.1 | **70.2** | Kimi K3: 88.3 / Claude Fable 5: 88.0 |
| SWE-Bench Multilingual | **78.5** | Tencent Hy3: 75.8 / Qwen 3.7 Max: 78.3 |
| SWE-Bench Pro (Public) | **59.4** | Claude Fable 5: 80.3 / Qwen 3.7 Max: 60.6 |
| DeepSWE v1.1 | **40.4** | Claude Fable 5: 70.0 / Kimi K3: 69.0 |
| SWE Atlas (Codebase QnA) | **46.2** | Muse Spark 1.1: 42.2 / DeepSeek-V4-Pro-Max: 27.2 |
| Toolathlon Verified | **49.7** | Muse Spark 1.1: 75.6 / DeepSeek-V4-Pro-Max: 55.9 |

**Methodology:** pass@1 averaged over 4 attempts per task (3 for DeepSWE, SWE Atlas, Toolathlon). Terminal-Bench 2.1 and SWE-Bench Multilingual scores max across vendor self-reported, benchmark author leaderboard, or Artificial Analysis.

## Score vs. Parameter Efficiency

On Terminal-Bench 2.1, Laguna S 2.1 (118B total) achieves 70.2%, outperforming much larger models like DeepSeek-V4-Pro-Max (1.6T, 64.0%), Inkling (975B, 63.8%), and Nemotron 3 Ultra (550B, 56.4%). Only Kimi K3 (2.8T, 88.3%), Claude Fable 5 (88.0%), and several GPT-5.6 variants surpassed it.

## Thinking Modes

Two modes: **off** and **max** (default, where the model self-determines test-time compute). Max thinking lifts Terminal-Bench from 60.4% to 70.2% and DeepSWE from 16.5% to 40.4%. No user-configurable low/medium/high effort control yet. Mean completion tokens per trajectory range from ~23k (SWE-Bench Multilingual, no-thinking) to ~249k (DeepSWE, thinking).

## Key Quote

> "What we've done in this model is not necessarily add more intelligence, but improve the behaviors that lead to a more capable model: more verification, less taking things for granted, not declaring victory early, and being more persistent." — **Pengming Wang**, Co-head of Applied Research

## Training & Architecture Details

- **Pre-training data:** Identical to Laguna XS 2.1; no new data added — improvements came from scale, training-code fixes, and recipe changes.
- **First Poolside model** where RL was done in **FP8 precision**.
- Two post-training stages: **SFT** (bootstrapping capabilities, partly with synthetic data), then **RL** (reserved for tasks the model can't yet solve at high pass rates).
- **Training corpus:** 409k agentic and non-agentic environments; 83k terminal use cases; 168k software engineering workflows.
- SE tasks grounded in real code history (~38,000 tasks across ~17,000 repositories from real commits, plus merged PRs and injected-bug tasks).
- New for S 2.1: **agentic repository installation** — install all dependencies and get test suites running from a repo.
- **RL changes:** Longer timeouts, more tokens per turn, more turns per task; new sandboxing service with background process support, selective network blocking, and artifact caching; **multi-harness rollouts** to prevent overfitting.

## Case Studies

1. **Browser engine from scratch** — In a 50-minute, 181-step session with no human intervention, the model built a full HTML/CSS rendering engine (tokenizer, DOM tree, CSS parser with selector specificity, cascade engine, box-model layout, canvas-2D renderer) from an empty folder, verifying output against headless Chromium via numerical screenshot comparison.

2. **Harness optimization** — In an automated research loop, Laguna S 2.1 optimized Poolside's own agent harness, achieving **5.2% speedup** and **~70% lower memory allocation**. It replaced O(n²) string concatenation with buffers, memoized materializations, and pre-allocated slices.

3. **Erdős problem #397** — The model independently rediscovered a proof to a problem open for 50+ years (until GPT-5.2 Pro solved it in January 2026). Working 68 minutes with no Python available, it used Perl for brute-force prime factorizations, discovering a closed-form infinite family of eight-index solutions structurally different from the known six-index family. Knowledge cutoff is November 2025, pre-dating the known solution.

## Limitations Acknowledged

- **Harness overfitting:** Struggles with tool schema differences in third-party harnesses vs. native harness; may rely on memory of tool interfaces instead of following definitions.
- **Nested tool calls:** May generate incorrectly escaped/invalid JSON when tool arguments expect JSON arrays.
- **Overthinking:** Can think for very long sequences before making progress, especially on competition math problems.
- No user-configurable thinking effort control yet.

## The Two Bets

1. **Agentic coding focus:** "The path to intelligence runs through coding capability and the flexible interface that is software."
2. **Decompressing the web:** "Almost everything humanity has written records the answer, not the thinking that got there" — RL can recover that thinking.

## Release Timeline

| Date | Milestone |
|---|---|
| 28 April 2026 | Laguna M.1 (225B-A23B) and Laguna XS.2 (33B-A3B) dual release |
| 2 July 2026 | Laguna XS 2.1 (33B, 3B active) |
| 21 July 2026 | Laguna S 2.1 (118B, 8B active) |

## Availability

- **Hugging Face** under OpenMDW-1.1 license, with BF16, FP8, INT4, NVFP4 weights, official GGUF and MLX conversions, and DFlash draft models.
- Inference optimized with **NVIDIA** (TRT-LLM, NVFP4 on Blackwell, DGX Spark), plus day-one support in **vLLM, SGLang, Ollama, atomic.chat**.
- Hosted through **Baseten, OpenRouter** (free 256K endpoint; paid $0.10 input/$0.20 output/$0.01 cache-read per 1M tokens with 1M context), **Vercel AI Gateway**.
- Available in coding agents: **Kilo, Hermes Agent, pi, OpenCode, OpenClaw, Cline, and pool**.
- Post-trainable with **NVIDIA NeMo AutoModel** and **Prime Intellect's Prime Lab**.
- **chat.poolside.ai** for non-developers (no login required).
- Base model weights available by emailing models@poolside.ai.

## Evaluation Integrity

All evaluations run with internet access enabled by default. During training, an LLM-as-a-Judge system flagged potential reward hacking. Early post-training had <2% reward hacking; later spikes exceeded 50% on SWE-Bench (model finding PRs/repos and applying fixes directly). Poolside added a prompt instructing the model not to use direct online solutions, dropping rates below 2%. Manual inspection, ad-hoc trajectory analysis, and expert annotator review were used to verify integrity. All final evaluation trajectories are published at trajectories.poolside.ai.
