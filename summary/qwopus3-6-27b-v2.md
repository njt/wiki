---
url: https://huggingface.co/Jackrong/Qwopus3.6-27B-v2
title: Qwopus3.6-27B-v2
author: Jackrong
date_published: 2026
date_fetched: 2026-08-07
topics:
  - ai-research-and-models
---

Qwopus3.6-27B-v2 is a community experimental fine-tune of Qwen3.6-27B by Jackrong (with hardware engineer Kyle Hessling), released on Hugging Face. It supports vision and tool-use capabilities and achieves 5,301 downloads in its first month.

The model's key innovation is **Trace Inversion** — a distillation technique for extracting reasoning capability from closed models that don't expose their chain-of-thought. A surrogate Trace-Inverter-4B model (trained on open-source reasoning traces compressed via Qwen3-235B into "reasoning bubbles") reconstructs Claude-4.7-Max's compressed outputs into full step-by-step reasoning chains. These synthetic traces are spliced into `<think>` tags and used to fine-tune the base Qwen3.6-27B.

Training uses a three-stage curriculum: Format Inception (<4K tokens, stable reasoning templates), Complexity Expansion (4K–8K, high-difficulty logic), and Long-Context SFT (8K–32K, multi-turn + 10% short-sample replay to prevent capacity drift). The model was fine-tuned on an NVIDIA DGX Cluster with H100s and RTX 6000 Pros using the Unsloth framework.

Known issues include LoRA weight-merge OOM risks and dependency compatibility problems between PEFT, Transformers 5.x, and Unsloth patches. The model card explicitly warns against unattended secondary fine-tuning without version pinning.
