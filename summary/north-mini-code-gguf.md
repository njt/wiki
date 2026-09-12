---
url: https://huggingface.co/heimann/North-Mini-Code-1.0-GGUF
title: North-Mini-Code-1.0-GGUF
author: heimann
date_fetched: 2026-07-05
date_published: unknown
topics:
  - ai-research-and-models
---

# North-Mini-Code-1.0-GGUF

GGUF quantizations of CohereLabs/North-Mini-Code-1.0, a 30B Mixture of Experts model (30B-A3B MoE) using the cohere2_moe architecture. Licensed under Apache 2.0.

## Important Usage Note

The model requires [speedy-llama](https://github.com/heimann/speedy-llama) until upstream llama.cpp gains cohere2_moe support. A pull request (PR #24260) is pending merge. The files "use the standard GGUF keys and are expected to load on upstream once that PR lands."

## Quantization Files & Benchmarks

| File | Size | Wikitext-2 PPL | Same top token as bf16 | 24GB card |
|------|------|---------------|------------------------|-----------|
| Q4_K_M | 18.6 GB | 8.34 (+3.2% vs bf16) | 90.4% (mean KLD 0.049) | ~230 tok/s, fully offloaded |
| Q5_K_M | 21.7 GB | 8.19 (+1.3% vs bf16) | 93.4% (mean KLD 0.023) | ~211 tok/s, fully offloaded (8K ctx) |

The bf16 baseline PPL is 8.09, measured over 64x512-token chunks of wikitext-2-raw test, KL-divergence computed against bf16 logits on the same tokens.

## Suggested Sampling Parameters

```
llama-cli -m north-mini-code-Q4_K_M.gguf --jinja -ngl 99 --temp 1.0 --top-p 0.95
```

## Other Details

- Downloads last month: 766
- Model size: 30B params
- Architecture: cohere2moe
- Chat template: Embedded (converted from the bf16 safetensors release)
- Tags: GGUF, code, Mixture of Experts, conversational
- Base model: CohereLabs/North-Mini-Code-1.0

## Supported Applications

Instructions provided for: llama.cpp, LM Studio, Jan, Ollama, Unsloth Studio, Pi, Hermes Agent, Atomic Chat, OpenClaw, Docker Model Runner, Lemonade, and llama-cpp-python.
