# TimesFM

TimesFM is Google Research's **decoder-only foundation model for time-series forecasting** — think "GPT for time series." A 200M-parameter transformer pretrained on massive time-series data that can forecast any univariate time series without task-specific training, handling context windows up to 16K steps with quantile uncertainty estimates. Ships as a pip-installable inference library in both PyTorch and JAX/Flax, with optional covariate support via in-context linear regression. Published at ICML 2024, now deployed in BigQuery ML, Google Sheets, and Vertex AI.

#tool #project #forecasting #time-series #transformer #foundation-model

---

## Architecture

TimesFM is a **decoder-only transformer** — the same architectural family as GPT — applied to time series data instead of text. The core insight is treating time series forecasting as next-patch prediction rather than next-token prediction.

### Model Structure

- **200M parameters**: 20 transformer layers, 16 attention heads, 1280 model dimensions (80 per head)
- **SiLU/Swish activations** throughout, **RMSNorm** for all normalization
- **Rotary Position Embeddings** (RoPE) for positional encoding
- **Fused QKV projection** — a single linear layer projects to query, key, and value simultaneously, reducing memory and compute
- **10 quantile outputs**: mean forecast plus 9 quantiles (0.1 through 0.9) for uncertainty

The architecture is defined once in a framework-agnostic frozen dataclass (`TimesFM_2p5_200M_Definition` in `src/timesfm/timesfm_2p5/timesfm_2p5_base.py:84-132`) and compiled to both PyTorch (`timesfm_2p5_torch.py`) and Flax (`timesfm_2p5_flax.py`) from the same config.

### Tokenization: Patching, Not Text

Instead of tokenizing text into subwords, TimesFM divides time series into **fixed-length patches**:

- **Input patches**: 32 consecutive time steps form one "token"
- **Output patches**: 128 time steps per generated patch (4× the input resolution)
- A simple `ResidualBlock` (two-layer MLP with residual skip, `src/timesfm/torch/dense.py:23-56`) serves as the "tokenizer," concatenating each patch with its binary mask

This patching approach is computationally efficient — the model processes 32× fewer tokens than a per-point approach — at the cost of losing fine-grained patterns within a single patch.

### Inference Pipeline

The `decode()` method (`timesfm_2p5_torch.py:115-219`) runs three phases:

1. **Prefill**: Input is divided into patches. Running mean/variance is computed patch-by-patch for Reversible Instance Normalization (RevIN, `src/timesfm/torch/util.py:77-94`). Normalized patches + masks feed through the full transformer.
2. **Autoregressive generation**: The model's own output (last patch) becomes the next input. Running statistics update continuously. A KV-cache (`DecodeCache` in `util.py:24-29`) prevents recomputation of previous patches.
3. **Post-processing**: Outputs are renormalized, flip-invariance is enforced, quantile crossing is fixed, and positivity constraints are applied.

The `compile()` pattern (`timesfm_2p5_torch.py:377-512`) pre-validates configuration (context/horizon must be patch-aligned), then wraps the decode function in a closure that handles preprocessing, optional `torch.compile`, and all the post-processing flags. In Flax, compilation uses `nnx.pmap` for multi-device parallelism.

---

## Key Techniques

### 1. Flip Invariance as Inference-Time Guarantee

The model guarantees `TimesFM(aX + b) = a × TimesFM(X) + b` for any scaling factor `a` — even negative. It does this **at inference time, not through training constraints**:

```python
# timesfm_2p5_torch.py:456-468
if fc.force_flip_invariance:
    # Run inference on negated inputs
    flipped_pf_outputs, flipped_quantile_spreads, flipped_ar_outputs = (
        self.model.decode(forecast_config.max_horizon, -inputs, masks)
    )
    # Flip the quantile dimension for negated inputs
    flipped_quantile_spreads = flip_quantile_fn(flipped_quantile_spreads)
    # Average the two passes
    quantile_spreads = (quantile_spreads - flipped_quantile_spreads) / 2
    full_forecast = (full_forecast - flipped_full_forecast) / 2
```

This doubles inference cost when enabled but eliminates sign-dependent behavior. For time series that can go negative (returns, differences), this is essential. For strictly positive series (sales, counts), it can be disabled.

### 2. Per-Dimension Scaled Attention (Replaces 1/√d)

Standard transformers divide attention scores by `√d_k`. TimesFM replaces this with a **learned per-dimension scale**:

```python
# torch/transformer.py:154-166
class PerDimScale(nn.Module):
    def forward(self, x):
        scale_factor = 1.442695041 / math.sqrt(self.num_dims) * F.softplus(self.per_dim_scale)
        return x * scale_factor
```

Initialized at zero (softplus(0) ≈ 0.693), each attention head dimension learns its own importance. Combined with `scale=1.0` in `F.scaled_dot_product_attention`, this gives the model finer-grained control over attention magnitude than the uniform 1/√d scaling.

### 3. Continuous Quantile Head

Quantile forecasts from a single projection layer often collapse — all quantiles converge to the median. TimesFM's solution is a **separate quantile head**:

- Main head: projects 1280-dim embeddings → 1280-dim point forecasts per output patch
- Quantile head: projects 1280-dim embeddings → 10240-dim (1024 output steps × 10 quantiles)

When `use_continuous_quantile_head=True`, intermediate quantiles are computed as **spread offsets from the median**:

```python
# timesfm_2p5_torch.py:471-476
full_forecast[:, :, quantile_index] = (
    quantile_spreads[:, :fc.max_horizon, quantile_index]
    - quantile_spreads[:, :fc.max_horizon, 5]  # subtract median spread
    + full_forecast[:, :fc.max_horizon, 5]      # add point-forecast median
)
```

This prevents the quantile head from "forgetting" the median location — quantiles can't drift far from the point forecast. The separate 30M-parameter head (vs 200M main model) provides dedicated capacity for uncertainty estimation.

### 4. In-Context Covariate Regression (XReg)

TimesFM supports covariates without retraining by fitting a **batched linear model in context**:

```python
# utils/xreg_lib.py:72-521
class BatchedInContextXRegLinear:
    def fit(self, ridge=0.0, ...):
        # 1. Build design matrix: numerical (standardized) + categorical (one-hot)
        # 2. Pad to power-of-2 for JAX efficiency
        # 3. Solve via jnp.linalg.pinv with optional ridge penalty
        beta_hat = jnp.linalg.pinv(X_train.T @ X_train + ridge * eye) @ X_train.T @ y
        # 4. Predict on test horizon
        y_hat = X_test @ beta_hat
```

Two modes:
- **"xreg + timesfm"**: Fit linear model on covariates → forecast residuals with TimesFM
- **"timesfm + xreg"**: Forecast with TimesFM → fit linear model on residuals → adjust

The clever part is that covariates are **batched**: different time series can have different lengths, different categorical levels, and different covariate sets — the XReg library handles ragged batching, one-hot encoding, standardization, and optional subsampling (`max_rows_per_col`) for performance.

### 5. Reversible Instance Normalization (RevIN) with Streaming Stats

Normalization is critical for time series with varying scales. TimesFM uses **streaming RevIN** during autoregressive decode:

```python
# torch/util.py:33-74
def update_running_stats(n, mu, sigma, x, mask):
    # Welford-style online algorithm: update mean and variance
    # from a new batch of observations while accounting for masks
    new_n = n + inc_n
    new_mu = (n*mu + inc_mu*inc_n) / new_n_safe
    term1 = n * sigma^2
    term2 = inc_n * inc_sigma^2
    term3 = n * (mu - new_mu)^2
    term4 = inc_n * (inc_mu - new_mu)^2
    new_var = (term1 + term2 + term3 + term4) / new_n_safe
```

During autoregressive decode, each new output patch updates the running statistics, ensuring the model always normalizes relative to the full history it has seen so far — not just the original input context.

### 6. QK Normalization

Both query and key tensors get independent RMSNorm before attention (`torch/transformer.py:207-212`). This technique, borrowed from recent LLM research, stabilizes attention score computation and prevents the attention distribution from collapsing to a single position during long autoregressive sequences.

---

## Design Decisions

### What They Optimized For

- **Inference speed over training flexibility**: No training code is open-sourced. The entire repo is an inference library with `torch.compile`, `nnx.pmap`, and fused QKV — all inference-time optimizations.
- **API simplicity over modeling flexibility**: `model.forecast(horizon, inputs)` with a single `ForecastConfig` dataclass for all flags. Compare to statsmodels or Prophet where you configure seasonality, trend changepoints, holidays individually.
- **Patch efficiency over point-level precision**: 32-step patches give ~32× compute reduction vs per-point processing. A fair trade for most business forecasting where sub-32-step precision is rarely needed.
- **Framework portability**: Same model definition runs on both PyTorch and JAX/Flax. The Flax version uses `nnx.pmap` for TPU/GPU parallelism; PyTorch targets single-GPU/CPU inference.

### What They Sacrificed

- **No training code**: You can't fine-tune the model architecture, only run inference. Fine-tuning is limited to LoRA adapters via HuggingFace PEFT (shown in examples but not part of the core library).
- **Univariate only (natively)**: The model forecasts one series at a time. Multivariate relationships must be captured via the XReg covariate system, which uses linear regression — powerful but limited compared to a natively multivariate model.
- **Removed frequency indicator**: v2.0 required specifying `frequency` (hourly, daily, etc.); v2.5 dropped it. The model must infer periodicity from data, which means it needs to see at least one full seasonal cycle in the context window to capture it.
- **Fixed patch alignment**: Context and horizon must be multiples of patch sizes (32 and 128). The `compile()` method silently rounds up, which can waste context capacity.
- **No probalistic model**: Quantiles provide uncertainty but there's no full predictive distribution. You get 9 points on the CDF, not a parametric distribution you can sample from.

### The "Decoder-Only" Choice

Using a decoder-only architecture (rather than encoder-decoder like T5, or full transformer) is a deliberate simplification. The model only needs to look backward in time (causal attention), which maps naturally to forecasting. Encoder-decoder would add parameters and complexity for bidirectional context that doesn't exist in pure forecasting. The trade: the model can't "fill in" missing values mid-series as naturally as a bidirectional model could.

---

## Comparison Notes

- **vs Prophet (Meta)**: Prophet requires per-series configuration (seasonality, changepoints, holidays) and fits one model per series. TimesFM is a single pretrained model that works zero-shot on any series — fundamentally a "foundation model" approach vs "fit a model per series."
- **vs ARIMA/ETS (classical stats)**: Classical methods are parametric and interpretable but require stationarity, handle limited seasonality, and can't leverage patterns learned across millions of series. TimesFM trades interpretability for scale and generality.
- **vs Chronos (Amazon)**: Both are pretrained time series transformers. Chronos uses a T5-style encoder-decoder and tokenizes by binning values into a fixed vocabulary. TimesFM uses decoder-only with patch tokenization — more similar to how GPT handles text. Chronos is larger (200M–710M params vs TimesFM's 200M) but TimesFM supports longer context (16K vs Chronos's 512).
- **vs Lag-Llama**: Lag-Llama uses lag features as tokens in a decoder-only model. TimesFM's patching approach is simpler — no need to choose which lags matter — but potentially less interpretable.
- **vs TimeGPT (Nixtla)**: TimeGPT is a commercial API with a similar "one model forecasts everything" promise. TimesFM is open-source, runs locally, and the Flax version can leverage TPU parallelism. TimeGPT has a more polished API but you can't inspect, modify, or self-host it.

---

*Source: https://github.com/google-research/timesfm · Paper: "A decoder-only foundation model for time-series forecasting" (arXiv:2310.10688, ICML 2024) · Fetched 2026-07-03*
