---
title: "GPU Memory Calculator for Self-Hosted LLM Inference"
url: https://selfhostllm.org/
author: Eran Sandler
date_fetched: 2026-05-18
date_published: 2026
section: "LLMs"
topics:
  - local-and-open-source-inference
  - ai-infrastructure-and-hardware
---

# GPU Memory Calculator for Self-Hosted LLM Inference

An interactive calculator and reference tool for mapping GPU hardware to LLM inference performance. Answer the question: "If I have this hardware, what models can I run, how many concurrent requests can I handle, and how fast will tokens generate?"

## Hardware Configuration

GPU Model options: RTX 4090 (24GB), RTX 4080 (16GB), RTX 4070 Ti (12GB), RTX 4070 (12GB), RTX 4060 Ti 16GB, RTX 4060 Ti 8GB, RTX 3090 Ti (24GB), RTX 3090 (24GB), RTX 3080 Ti (12GB), RTX 3080 (10GB), A100 40GB, A100 80GB, H100 (80GB), V100 32GB, RTX 6000 Ada (48GB), RTX 6000 Pro Blackwell Workstation (96GB), RTX 6000 Pro Blackwell Max-Q (96GB), RTX 6000 Pro Blackwell Server (96GB), RTX 5000 Pro Blackwell (48GB), RTX 5000 Pro 72GB Blackwell (72GB), RX 7900 XTX (24GB), RX 7900 XT (20GB). Also configurable: number of GPUs, VRAM per GPU, system overhead in GB.

## Model Configuration

Extensive model database with memory estimates and MoE active/total parameter breakdowns. Includes: Llama 3/4 families, Qwen/Qwen2/Qwen3/Qwen3.5 families, Kimi K2/K2.5, MiniMax M1/M2/M2.1/M2.5, Nemotron variants, DeepSeek V3/R1 families, Mistral/Mixtral families, Codestral, GPT-OSS, GLM-4/4.5/4.6/4.7 families, Gemma 2/3, Phi-3, Granite 4.0, OLMo 2/3, Yi, CodeLlama, Falcon, and more.

Quantization options: FP16/BF16 (Full Precision), INT8 (~25% reduction), INT4 (~50% reduction), MXFP4 (~70% reduction), Extreme Quant (~75% reduction).

Context length presets from 1K to 2M tokens.

## How Max Concurrent Requests is Calculated

**Formula: Max Concurrent Requests = Available Memory / KV Cache per Request**

Step-by-step breakdown:
1. **Total VRAM Available** = Number of GPUs x VRAM per GPU
2. **Model Memory (Adjusted for Quantization)** = Base Model Memory x Quantization Factor
3. **KV Cache per Request** = (Context Length x Adjusted Model Memory x KV Overhead) / 1000
4. **Available Memory for Inference** = Total VRAM - System Overhead - Model Memory
5. **Maximum Concurrent Requests** = Available Memory / KV Cache per Request

What the results mean:
- < 1 request: Can't handle full context length; need smaller context or better GPU
- 1-2 requests: Basic serving capability, suitable for personal use
- 3-5 requests: Good for small-scale deployment
- 10+ requests: Production-ready for moderate traffic

## Mixture-of-Experts (MoE) Handling

MoE models (Mixtral, DeepSeek V3/R1, Qwen3 MoE, Kimi K2/K2.5, GLM-4.7) work differently: Total Parameters is the full model size, but only a subset of experts are used per token (Active Parameters). Memory calculation uses active memory only. Example: Mixtral 8x7B shows "~94GB total, ~16GB active" -- calculate using 16GB.

## How GPU Performance is Estimated

**Formula: Tokens/sec = (Memory Bandwidth / Model Size) x Efficiency x Quantization Boost x Context Impact**

Key factors:
1. **Memory Bandwidth**: RTX 4090: 1 TB/s, A100: 1.6-2 TB/s, H100: 3 TB/s
2. **Model Size Efficiency**: Smaller models utilize bandwidth better (<=7B: ~85%, 7-30B: ~70%, 30-70B: ~50%, 70B+: ~30%)
3. **Quantization Speed Boost**: INT3: 2.5x, INT4: 2.2x, INT8: 1.3x, FP16: 1.0x (baseline)
4. **Context Length Impact**: More tokens = more memory ops (<=8K: 85%, 8-32K: 60%, 32-128K: 30%)
5. **Multi-GPU Scaling**: Not perfect due to communication overhead (2 GPUs: ~85%, 4 GPUs: ~75%, 8 GPUs: ~65%)

Performance Ratings:
- Excellent (>100 tok/s): Real-time conversation, instant responses
- Good (50-100 tok/s): Smooth interaction, minimal wait
- Moderate (25-50 tok/s): Acceptable for most uses, some waiting
- Slow (10-25 tok/s): Noticeable delays, patience required
- Very Slow (<10 tok/s): Long waits, consider optimization

Important caveats: Framework matters (vLLM, TGI, Ollama have different optimization levels), batch size affects throughput, first token is slower than generation, real-world variance ~±20%, and professional GPUs (A100/H100) have better memory subsystems than consumer cards.
