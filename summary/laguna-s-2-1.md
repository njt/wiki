---
url: https://poolside.ai/blog/introducing-laguna-s-2-1
title: "Introducing Laguna S 2.1"
author: Poolside team
date_fetched: 2026-07-25
date_published: 2026-07-21
---

Laguna S 2.1 is Poolside's 118B-total-parameter Mixture-of-Experts coding model
(8B active per token), released July 2026 with a 1M-token context window. Trained
in under nine weeks on 4,096 NVIDIA H200 GPUs, it scores 70.2% on Terminal-Bench
2.1 with thinking enabled — outperforming much larger models like
DeepSeek-V4-Pro-Max (1.6T params, 64.0%) and Nemotron 3 Ultra (550B, 56.4%).

The model supports two thinking modes (off and max), with max thinking lifting
DeepSWE scores from 16.5% to 40.4%. It runs on a single NVIDIA DGX Spark,
targeting local-machine agentic coding as its niche. RL post-training was done
in FP8 precision — a first for Poolside — across 409k agentic and non-agentic
environments, with new multi-harness rollouts to prevent overfitting.

Three case studies demonstrate capability: building a full HTML/CSS rendering
engine from scratch in a single 50-minute session, optimizing Poolside's own
agent harness for a 5.2% speedup, and independently rediscovering a proof to
Erdős problem #397 using only Perl for brute-force work. Acknowledged
limitations include harness overfitting (relying on memory of tool interfaces
rather than reading schemas), nested-tool-call JSON errors, and occasional
overthinking on competition math.

The release is available on Hugging Face under OpenMDW-1.1, with day-one
inference support in vLLM, SGLang, Ollama, and TRT-LLM, plus hosted access via
OpenRouter and Baseten.
