---
url: https://huggingface.co/RecursiveSI/flint
title: "Recursive Flint"
author: Recursive
date_fetched: 2026-10-24
date_published: 2026-10-24
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

Recursive's model card for Flint, a mixture-of-experts language model built specifically for coding agents: 39.8B total parameters with about 2.5B active per token (the "40B-A2.5B" designation), decoder-only MoE with 256 routed experts (6 selected per token) plus a shared expert, 49 layers mixing 35 sliding-window attention layers (513-token window) with 14 full-attention layers, and a documented 262K-token context. It is trained from scratch through pretraining, mid-training, SFT, and RL, with agentic tasks in the mix.

The release evaluations are agentic-benchmark-heavy: 54.9% on Terminal-Bench 2.1, 89.7% on SWE-bench Pro v2, and 74.2% on SWE-bench Verified, using mini-swe-agent as the harness and averaging pass@1 over at least five attempts. Notably, Recursive counts solutions obtained via web retrieval as failures on Terminal-Bench — an adjustment developer-reported comparison scores may not apply, which the card flags openly.

The HF repo ships a BF16 checkpoint, tokenizer, chat template, and an SGLang serving kit (~80 GB download); it runs one B300 or H200 GPU per replica. Quantized variants for NVIDIA and Apple Silicon are prepared separately. Transformers modeling code is not yet included. Recommended settings are temperature 1.0, top-p 0.95, top-k 20, and at least a 32,768-token output budget for hard problems, with `reasoning_content` preserved in conversation history across multi-turn tool use.

The card is unusually candid about limitations: the model can emit faulty code or inappropriate tool calls, and it explicitly warns that structured tool-call grammar does not prevent unsafe actions — tool-using applications must enforce their own permission and execution boundaries. Apache 2.0 licensed; knowledge cutoff June 2026.
