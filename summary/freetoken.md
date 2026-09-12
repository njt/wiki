---
url: https://github.com/FlashML-org/FreeToken
title: FreeToken
author: FlashML-org (Shuo Yang, Xiaoze Fan, Melissa Pan, Haocheng Xi, Zhe Wang, Shanlin Sun, Kurt Keutzer, Song Han, Matei Zaharia, Chenfeng Xu, Ion Stoica)
date_fetched: 2026-08-22
date_published: 2026
topics:
  - local-and-open-source-inference
---

# FreeToken — Repository Analysis

## Overview

FreeToken is an edge-native Mixture-of-Experts (MoE) serving engine for running frontier-scale open-weight models (290B+ total parameters) on hardware you already own — gaming PCs, laptops, and workstation GPUs (NVIDIA RTX 30/40/50). Its framing is that VRAM is not a hard boundary: the GPU, CPU, host RAM, and PCIe interconnect together form one elastic inference platform, and the engine's job is to schedule work across them by measured bandwidth rather than by a fixed device layout.

The headline technique is **bandwidth-adaptive CPU–GPU co-execution** (the "$q^\star$ policy"). All expert weights live in pinned host-RAM banks and stream into a unified GPU slot cache governed by a global LRU. At decode time a fraction of each step's cache misses is fetched over PCIe while the overflow is computed on the CPU by a C++ `CpuMoeExecutor`; a `benchbw` profiler measures real CPU-MoE vs PCIe-gather kernels and recommends the hybrid backend only when CPU bandwidth exceeds 2× PCIe bandwidth, sizing the fetch fraction so fetch and compute finish together.

Supporting machinery: full-layer double-buffered prefill streaming that splits resident experts (device-to-device gather) from PCIe misses; the FTW checkpoint format (every tensor 4096-aligned, 8 GiB shards, chunked multi-threaded O_DIRECT reads); semantic anchor checkpoints that freeze recurrent/KV state at the tool-call opener so agentic context edits (tool calls, thinking blocks) skip recompute; and in-place elastic re-allocation of VRAM between expert cache and KV memory with CUDA-graph re-capture and rollback.

The project is Apache 2.0, released alongside a paper (arXiv:2608.16157, "FreeToken: Efficient Edge-Native MoE Serving with Bandwidth-Adaptive Execution"), and builds on design and code from SGLang, vLLM, FlashInfer, flash-linear-attention, LightLLM, and llama.cpp.

## What it runs

Frontier open-weight MoE models across parameter scales and quantization formats: DeepSeek-V4-Flash, Qwen3.x (including Qwen3.5-MoE and Qwen3.6-35B-A3B), GLM-5.2 / GLM4-MoE, Gemma 4, GPT-OSS, MiniMax M2/M3, Llama, Mistral, and Muse Glimmer. Quant formats span bf16, fp8_block, q4_0 (GGUF), nvfp4, mxfp4_triton, and ds_fp4. It exposes Anthropic/OpenAI-compatible APIs so coding and tool-calling agents (Codex, Claude Code, OpenCode, OpenClaw, DeepSeek Harness) can point at it directly, and ships a desktop GUI plus a `ft` CLI.

## Key components (from the repository)

- `engine/engine.py` — the Engine: builds the model on the `meta` device, loads weights, initializes the MoE offload cache, KV pool, page table, attention/MoE backends, sampler, and CUDA-graph decode runner. `_adjust_config` reconciles config across dense/MoE and per-model page-size constraints.
- `moe/offload_cache.py` — the expert slot cache: `_BANK_SCHEMAS` declares each quant format's bank layout; `_build_copy_plan` emits fused multi-bank copy descriptors; `ensure_experts` / `ensure_experts_hybrid` materialize experts.
- `moe/benchbw.py` — measures real CPU-MoE GEMV vs PCIe gather (and contended/overlapped bandwidth), then recommends hybrid vs GPU-only.
- `scheduler/scheduler.py` + `scheduler/cache.py` — continuous-batch scheduler with overlap scheduling, tool-call anchor detection, and polymorphic radix KV caches (hybrid / sliding-window / plain).
- `checkpoint/ftw.py` — the FTW weight format writer/reader with O_DIRECT reads and mmap fallback.

## Why it matters

FreeToken is the same MoE-offload-and-attack-the-bandwidth-term thesis that [[MiMo-V2.5-Pro-UltraSpeed]] demonstrated on an 8-GPU node, brought down to a single consumer GPU by leaning on CPU compute and host RAM as first-class execution resources instead of treating them as slow fallbacks.
