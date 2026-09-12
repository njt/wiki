---
url: https://magazine.sebastianraschka.com/p/recent-developments-in-llm-architectures
title: "Recent Developments in LLM Architectures: KV Sharing, mHC, and Compressed Attention"
author: Sebastian Raschka
date_fetched: 2026-05-22
date_published: 2026-05-16
topics:
  - ai-research-and-models
---

## Context

Raschka surveys open-weight LLM releases from early 2026, focusing exclusively on architecture design — skipping datasets, training recipes, RL, and benchmarks. The unifying theme: making long-context inference cheaper for reasoning models and agent workflows.

## Models Covered

### Gemma 4 (Google)
- **Cross-Layer KV Sharing**: Later layers reuse KV projections from earlier layers of the same attention type (sliding-window or full-attention). E2B: 35 layers, 15 compute KV, 20 share. ~50% KV cache reduction.
- **Per-Layer Embeddings (PLE)**: Additional capacity stored in per-layer embedding lookup tables rather than wider attention or FFN weights. The "E" in E2B/E4B = "effective" parameter count (closer to main transformer compute cost).

### Laguna XS.2 (Poolside)
- **Per-Layer Attention Budgeting**: Different query-head counts per layer while KV heads stay fixed at 8. Full-attention layers get 6 Q-heads/KV-head; sliding-window layers get 8 Q-heads/KV-head. More capacity where it's cheaper, less where it's expensive.
- 40 layers: 30 sliding-window (512 tokens) + 10 global attention.

### ZAYA1-8B (Zyphra)
- **Compressed Convolutional Attention (CCA)**: Compresses Q, K, and V, then performs attention directly in compressed latent space. Unlike MLA (which compresses only KV per-token for cache savings), CCA compresses all three and uses convolutional mixing on compressed Q/K to recover expressiveness. Reduces both KV cache size AND attention FLOPs.
- Trained on AMD GPUs.
- Extreme MoE: one active expert per token.

### DeepSeek V4 (DeepSeek)
- **Manifold-Constrained Hyper-Connections (mHC)**: Replaces the single residual stream with 4 parallel streams and learned, constrained mappings between them. Constraints (doubly stochastic matrices, non-negative parameters) prevent the signal amplification/cancellation issues of unconstrained hyper-connections. Only 6.7% training overhead.
- **CSA + HCA**: Three attention branches per layer — sliding window (128 tokens), Compressed Sparse Attention (4:1 compression, sparse top-k), Heavily Compressed Attention (128:1 compression, dense). At 1M-token context: V4-Pro uses 27% FLOPs and 10% KV cache vs. V3.2 baseline.

## Key Observations

- Transformer block remains the foundation — targeted upgrades, not replacement
- Qualitative performance still driven by data quality/quantity and training recipes
- These architectural tweaks add ~10x code complexity vs. a basic transformer
- The pattern: all four models attack long-context cost from different angles
