---
url: https://huggingface.co/RadixArk/DaisyRose-27B
title: "DaisyRose-27B"
author: RadixArk
date_fetched: 2026-11-20
date_published: unknown
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

RadixArk's first model release: DaisyRose-27B, a multimodal reasoning model built on Qwen3.8-27B using **reinforcement learning only** — no supervised fine-tuning, no distillation. It was trained on about 1,200 curated long-horizon agent tasks (sessions up to 1M tokens, up to 1.5 hours in a sandbox, 100+ tool calls per attempt) for 18 days on 144 H200s (~63,000 GPU-hours).

The headline result is Terminal Bench 4.1 pass@1 rising from 5.6 (base Qwen3.8-27B) to 12.0, with gains on most coding, reasoning and instruction-following benchmarks. The comparison table includes much larger models (Inkling 2 Small at 594B total, Muse Spark Open at 816B) and GPT-6 Luna; the card is unusually transparent about evaluation methodology — repetitions per benchmark, harness used (mini-swe-agent, Terminus 2), and where each competitor number comes from.

The model keeps the base model's architecture, tokenizer and chat template, its image understanding, and its 256K context, positioning it as a drop-in replacement for Qwen3.8-27B. Weights ship under Apache-2.0 in BF16 (52 GB), FP8 (29 GB) and NVFP4 (23 GB), runnable on a single GPU from H200/B200 down to RTX 5090 and DGX Spark. Serving is recommended with SGLang plus DSpark speculative decoding; the card includes full launch commands, a quickstart with the recommended thinking-mode sampling settings (temperature 1.0, top_p 0.95, top_k 20), and guidance on preserving `reasoning_content` across multi-turn chats.

Training infrastructure: fully asynchronous RL with RadixArk's open-source Miles framework — 128 H200s running the agent on sandboxed tasks (served by SGLang, sandboxed on AgentENV), 16 more training on finished trajectories, weights syncing back each step. Tasks were built with Eigent AI, each targeting a specific base-model mistake and tuned so solve rate is neither too easy nor too hard; every task was reviewed alongside the model's attempts and fixed or removed if it could be gamed without real solving.
