---
url: https://github.com/google-research/timesfm
title: "TimesFM: A Decoder-Only Foundation Model for Time-Series Forecasting"
author: "Rajat Sen, Yichen Zhou, Abhimanyu Das, Petros Mol, Michael Chertushkin (Google Research)"
date_fetched: 2026-07-03
date_published: 2023-10 (arXiv), 2024 (ICML)
topics:
  - misc
---

# Full Analysis

## Repository Structure

```
src/timesfm/
  __init__.py                          # Package init, conditional imports for torch/flax
  configs.py                           # Framework-agnostic dataclass configs
  timesfm_2p5/
    timesfm_2p5_base.py                # Abstract base with forecast/compile interface
    timesfm_2p5_torch.py               # PyTorch 200M implementation (513 lines)
    timesfm_2p5_flax.py                # JAX/Flax 200M implementation (615 lines)
  torch/
    transformer.py                     # MultiHeadAttention, RotaryPE, Transformer (371 lines)
    dense.py                           # ResidualBlock, RandomFourierFeatures
    normalization.py                   # RMSNorm
    util.py                            # DecodeCache, update_running_stats, revin
  flax/
    transformer.py                     # Flax equivalents of all transformer layers (357 lines)
    dense.py                           # Flax ResidualBlock
    normalization.py                   # Flax RMSNorm
    util.py                            # Flax DecodeCache, revin, scan_along_axis
  utils/
    xreg_lib.py                        # Exogenous regressor library (521 lines)
tests/
  test_model_loading.py
  test_torch_utils.py
  test_torch_layers.py
  test_base_utils.py
  test_configs.py
v1/                                    # Archived v1/v2 code
timesfm-forecasting/                   # Agent skill and examples
```

## Architecture Deep Dive

### Model Architecture (TimesFM 2.5)

The model is a **decoder-only transformer** with these specifications:
- 200M parameters (down from 500M in v2.0)
- 16K context length (up from 2048)
- Input patch size: 32 time steps
- Output patch size: 128 time steps (ratio m=4)
- 20 transformer layers, 16 heads, 1280 model dims, 80 head dims
- SiLU/Swish activations throughout
- RMSNorm for all normalization
- Rotary Position Embeddings (RoPE)
- Fused QKV projection for efficiency
- 10 quantile outputs (mean + 9 quantiles: 0.1 to 0.9)

### Tokenization Scheme

Unlike LLMs that tokenize text, TimesFM "tokenizes" a time series by patching:
- Input: `input_patch_len = 32` consecutive time steps form one "token"
- Output: `output_patch_len = 128` time steps per output patch
- A simple ResidualBlock (input→hidden→output with residual connection) serves as the "tokenizer"
- The tokenizer concatenates the time series values with a binary mask indicating padding

### Inference Pipeline

1. **Prefill**: Input time series is divided into patches of 32 steps. Running mean/variance is computed patch-by-patch for Reversible Instance Normalization (RevIN).
2. **Forward Pass**: Normalized patches + masks → tokenizer → 20-layer transformer (with KV-cache for autoregressive decode) → output projections (point + quantile)
3. **Autoregressive Decode**: The model's own output (last output patch) becomes the input for generating the next set of patches. Running stats update continuously.
4. **Post-processing**: Renormalization, flip invariance enforcement, quantile crossing fix, positivity constraints

### Key Implementation Details

**Flip Invariance**: On by default. Runs inference twice — once on inputs, once on negated inputs — and averages. This guarantees `TimesFM(aX + b) = a * TimesFM(x) + b` for both positive and negative scaling. Negated-input quantiles are flipped before averaging.

**Continuous Quantile Head**: A separate 1280→10240 projection layer produces quantile "spreads" for up to 1024 time steps. When enabled, intermediate quantiles are computed as `spread[quantile] - spread[median] + point_forecast[median]`, preventing quantile collapse/crossing from the projection.

**Per-Dimension Scaled Attention**: Each attention head dimension has a learnable scale parameter (initialized at zero, activated via softplus, multiplied by `1.4427/sqrt(d)`). Combined with `scale=1.0` in dot-product attention (not standard `1/sqrt(d_k)`), this replaces the conventional scaling factor with a learned per-dimension alternative.

**QK Norm**: Query and key tensors get independent RMSNorm before attention computation, a technique borrowed from recent LLM training stability improvements.

**KV Cache for Inference**: Uses a `DecodeCache` dataclass (key, value, next_index, num_masked) passed through all 20 transformer layers. During autoregressive decode, new key/value entries are appended to the cache via slice assignment. The attention mask uses `num_masked` to ensure padding patches don't attend to real data and vice versa.

**JAX pmap**: The Flax implementation uses `nnx.pmap` with `axis_name="global_batch"` to parallelize across multiple TPU/GPU devices. Pre/post-processing functions (`_before_model_decode`, `_after_model_decode`) handle tiling and untiling with `einshape`.

**XReg Covariate Pipeline**: Supports four covariate types (dynamic numerical, dynamic categorical, static numerical, static categorical). Fits a linear model (ridge regression via `jnp.linalg.pinv`) on the context, then uses TimesFM either to forecast the residuals ("xreg + timesfm") or to provide a baseline that gets adjusted ("timesfm + xreg"). One-hot encodes categoricals, standardizes numericals, pads to power-of-2 for JAX efficiency.

## Key Innovations

1. **Patching as tokenization**: Applies LLM tokenization concepts to continuous time series
2. **Flip invariance as inference-time post-processing**: Not a training constraint, but a runtime guarantee
3. **Continuous quantile head**: Avoids quantile crossing without complex architectural changes
4. **Framework-agnostic configs**: Same model definition compiles to PyTorch and Flax
5. **Torch compile + pmap integration**: Both frameworks get framework-native compilation

## Design Trade-offs

- **Decoder-only**: Simpler than encoder-decoder, sufficient for autoregressive forecasting, but can't naturally handle bidirectional context
- **Removed frequency indicator**: v2.5 drops the explicit frequency input that v2.0 required. The model must learn periodicity from data alone. Simplifies the API but may require longer context for seasonal patterns
- **200M params (down from 500M)**: More efficient inference, but fewer parameters to capture complex patterns
- **No training code**: This is an inference-only open source release. Training methodology, data, and infrastructure remain proprietary
- **Fixed patch sizes**: Patching improves efficiency but means the model can't forecast at arbitrary granularity; fine-grained point-by-point patterns within a patch are lost
- **Unscaled attention with per-dim scaling**: Replaces the standard scaling convention with a learned approach, which may require careful initialization to avoid training instability

## Dependencies

- Core: numpy, huggingface_hub, safetensors
- PyTorch path: torch >= 2.0.0
- Flax path: flax, optax, einshape, orbax-checkpoint, jaxtyping, jax[cuda]
- XReg path: jax[cuda], scikit-learn
