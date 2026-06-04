---
url: https://thenewstack.io/jetbrains-mellum2-open-source-coding-model/
title: "JetBrains open-sources Mellum2 to go where Claude Code can't"
author: Paul Sawers
date_fetched: 2026-06-04
date_published: 2026-06-01
primary_source: https://blog.jetbrains.com/ai/2026/06/mellum2-goes-open-source-a-fast-model-for-ai-workflows/
primary_authors: Anton Semenkin, Nikita Pavlichenko (JetBrains)
---

# Mellum2 Goes Open Source: A Fast Model for AI Workflows

Source: JetBrains AI Blog, June 1, 2026. Authors: Anton Semenkin, Nikita Pavlichenko.

Note: The New Stack article by Paul Sawers could not be fully retrieved (paywall/interstitial). Content below is drawn from the JetBrains blog post (primary source) and web search results.

## What Is Mellum2?

Mellum2 is a 12-billion-parameter model trained from scratch by JetBrains, released as open source under Apache 2.0. It evolves from the original Mellum (4B dense, code completion only) into a full coding assistant capable of natural language interaction, code generation, tool calling, and multi-step agentic workflows.

## Architecture

- **Mixture-of-Experts (MoE):** 12B total parameters, 2.5B active per token. 64 experts, 8 active per token.
- **Grouped-Query Attention:** 4 KV heads for memory efficiency.
- **Sliding Window Attention:** Applied on 3 of every 4 layers.
- **Multi-Token Prediction (MTP) head:** Serves as both auxiliary pre-training objective and built-in draft model for speculative decoding.
- **Training:** Muon optimizer, FP8 hybrid precision, ~10.6 trillion tokens, three-phase curriculum.
- **Context window:** 131,072 tokens (up from 8,192 in original Mellum).
- **Not multimodal:** Trained exclusively on natural language and code data.

## Three Variants

All Apache 2.0:
1. **Base** — Foundation model
2. **Instruct** — Direct-answer mode for fast agentic commands
3. **Thinking** — Produces explicit reasoning traces before answering

## Performance

- **EvalPlus (thinking variant):** 78.4% — ahead of Qwen3.5-9B (71.8%) and Seed-Coder-8B (73.8%)
- **Single-request throughput:** Matches Qwen2.5-7B (~192 tok/s on one H100)
- **Under concurrent load:** 21% ahead of Qwen2.5-7B, 79% ahead of Qwen3-8B
- Competitive with open-weight models in 4B–14B range while running at per-token compute of a 2.5B dense model
- Benchmarks: LiveCodeBench, BFCL V4, AIME 2025/2026, GSM-Plus, GPQA Diamond, MMLU Redux, JetBrains Internal Pairwise, MixEval, IFEval

## Use Cases

1. **Routing and orchestration** — Analyzing prompts to select the right model or tool
2. **Low-latency RAG pipelines** — Context retrieval, summarization, response generation
3. **Fast sub-agents** — Breaking agent pipelines into discrete steps (context gathering, planning, validation)
4. **Private/local deployment** — Self-hosting for data sovereignty, air-gapped environments

## The "Focal Model" Philosophy

> "Frontier models will continue to push the limits, but practical AI products also require focal models: fast, specialized components that handle high-frequency tasks efficiently."

The thesis: "the future belongs to coordinated systems, not single models." Performance bottlenecks have shifted from raw capability to latency, throughput, and cost at scale. Many production steps are repetitive, latency-sensitive, and high-frequency — benefiting from a fast, routable model rather than a frontier general-purpose LLM.

## Why Open Source

> "Open source is how better tools get made."

Available on Hugging Face at the JetBrains collection page.

## Technical Report

Companion paper on arXiv: https://arxiv.org/abs/2605.31268
