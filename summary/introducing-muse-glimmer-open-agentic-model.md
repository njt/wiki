---
url: https://research.meta.ai/blog/introducing-muse-glimmer-open-agentic-model
title: Introducing Muse Glimmer: An Open Agentic Model That Runs on Your Device
author: Meta Superintelligence Labs
date_fetched: 2026-08-14
published: 2026-08 (undated in article)
tags: [model, agentic, local, open-weights, distillation]
---

Meta Superintelligence Labs open-sources **Muse Glimmer**, a 30-billion-parameter agentic model optimized for always-on local agent workflows, under a permissive Apache 2.0 license. It runs on a Mac or PC with a single consumer GPU and targets local agents, function calling, local coding, and LLM-as-a-judge evaluation.

The model is a distilled product of the closed [[Muse Spark]]: pre-training used logit distillation on Muse Spark's outputs, mid-training added longer-context agent-heavy data with richer reasoning traces, and post-training combined supervised fine-tuning with on-policy distillation and reinforcement learning across general, reasoning, coding, and agentic domains. It was evaluated under Meta's Advanced AI Scaling Framework.

Its advertised capabilities span end-to-end agentic task completion (DeepSearch QA, MCP-Atlas, τ-Bench, SWE-Bench), reliable tool use, multi-step reasoning, failure recovery, multimodal input via a perception encoder, scaffold compatibility (OpenClaw), controllable reasoning effort, and 100+ language support. Benchmarks are compared against Gemma4-31B and Qwen3.6-27B.

Local deployment is the differentiator: 4-bit quantization shrinks the model from 55+ GB to under 20 GB, fitting within a 24–32 GB memory envelope alongside KV cache, the perception encoder, and a DFlash-based speculative-decoding drafter. K-Quant-17GB speeds are reported on MacBook M4-Max, M5-Max, and RTX-5090. Distribution lands via Hugging Face, with llama.cpp/MLX/ExecuTorch integrations and partners including Ollama, LM Studio, Unsloth, vLLM, SGLang, Together AI, Fireworks AI, and OpenRouter.
