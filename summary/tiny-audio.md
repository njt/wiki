---
url: https://github.com/alexkroman/tiny-audio
title: "Tiny Audio"
author: Alex Kroman
date_fetched: 2026-10-11
date_published: 2026 (ongoing repo)
topics:
  - local-and-open-source-inference
  - ai-research-and-models
---

Tiny Audio is an open-source speech-to-text system built the cheap way: a frozen pretrained speech encoder joined to a frozen (or lightly LoRA-tuned) LLM decoder by a small trainable MLP projector. The published model reaches 1.84% WER on LibriSpeech test-clean and 7.42% pooled over 12 benchmarks (11,822 samples) while training only ~80M parameters, and the repo doubles as a free 3.5-hour course — a 5-minute laptop smoke test runs a real training loop before you rent any GPU.

The architecture is deliberately boring and entirely swappable from config: encoder → frame-stack + MLP projector → LLM decoder, all Hydra overrides. Two recipes ship: the published `granite_qwen_frozen` (Granite Speech 470M encoder, Qwen3.5 decoder, projector + LoRA) and a smaller `stage_1` teaching recipe (GLM-ASR-Nano, Qwen3-0.6B fully fine-tuned). The repo's real depth is in its measured engineering commentary: why the projector must NOT end in a trailing RMSNorm (it destroys per-token magnitude signal and loss went 3.166 vs 0.319), why dropout in a projector injects ~23% relative noise, and why LoRA at alpha/r=2.0 perturbed the frozen decoder harder (18.4% of weights) than fully fine-tuning it did (13.3%).

Beyond transcription it does word timestamps (Qwen3-ForcedAligner), speaker diarization (Nemotron-3 frame classifier with per-speaker masked-audio streams), token streaming, and a batched HTTP server (`ta serve`) that hits ~460x real time at 128 concurrent requests on an RTX 4090. `ta eval` reproduces the benchmark table and can score commercial APIs on the same samples; `ta runpod` wraps the whole train/serve/deploy loop on rented GPUs.
