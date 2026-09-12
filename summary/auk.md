---
url: https://huggingface.co/tencent/AuK
title: AuK — An Open-Source Foundation Model for Speech Generation and Editing
author: Tencent (Ziyang Ma, Zhikang Niu, et al.)
date_fetched: 2026-09-11
date_published: 2026-09-09
topics:
  - ai-research-and-models
---

AuK is Tencent's 1.5B-parameter open-source foundation model for speech generation and editing, trained on millions of hours of diverse audio data. It exposes every task through a single natural-language instruction interface: zero-shot and instruction-based TTS, content and acoustic editing, paralinguistic editing, speech enhancement, and source separation.

Two variants ship: **AuK** (the base model for high-quality generation) and **AuK-Flash** (a distilled model for fast 4-step inference). This repository holds the base model weights.

The supported task catalog spans four categories: **Speech Generation** (zero-shot TTS from a reference voice, instruct TTS from a voice description alone), **Content Editing** (rewrite what is said, rewrite lyrics while preserving melody and voice), **Acoustic Editing** (pitch, speed, volume), **Paralinguistic Editing** (emotion, timbre, de-accent, nonverbal sounds, whisper conversion), and **Enhancement & Separation** (speech enhancement, speech/music separation, target speaker extraction).

The checkpoint contains diffusion-transformer and layer-fusion weights; the MLLM encoder (Qwen2.5-Omni-3B) and VAE are loaded separately at runtime. SGLang-Omni shipped Day 0 support (`python -m sglang_omni.cli serve --model-path tencent/AuK`). The model is MIT-licensed, with a technical report at arXiv:2609.08936.
