---
url: https://github.com/google-research/timesfm
date_fetched: 2026-07-05
backfilled: true
---

TimesFM (Time Series Foundation Model) is a pretrained time-series foundation model developed by Google Research for time-series forecasting.

- Paper: A decoder-only foundation model for time-series forecasting, ICML 2024.
- All checkpoints: TimesFM Hugging Face Collection.
- Google Research blog.
- TimesFM in Google 1P Products:
- BigQuery ML: Enterprise level SQL queries for scalability and reliability.
- Google Sheets: For your daily spreadsheet.
- Vertex Model Garden: Dockerized endpoint for agentic calling.
 

This open version is not an officially supported Google product.

**Latest Model Version:** TimesFM 2.5

**Archived Model Versions:**

- 1.0 and 2.0: relevant code archived in the sub directory `v1`. You can`pip install timesfm==1.3.0`to install an older version of this package to load them.

Updated PyPI to `timesfm=2.0.2`. See Install.

Added fine-tuning example using HuggingFace Transformers + PEFT (LoRA) — see
`timesfm-forecasting/examples/finetuning/`.
Also added unit tests (`tests/`) and incorporated several community fixes.

Shoutout to @kashif and @darkpowerxo.

Huge shoutout to @borealBytes for adding the support for AGENTS! TimesFM SKILL.md is out.

Added back the covariate support through XReg for TimesFM 2.5.

TimesFM 2.5 is out!

Comparing to TimesFM 2.0, this new 2.5 model:

- uses 200M parameters, down from 500M.
- supports up to 16k context length, up from 2048.
- supports continuous quantile forecast up to 1k horizon via an optional 30M quantile head.
- gets rid of the `frequency`indicator.
- has a couple of new forecasting flags.

Since the Sept. 2025 launch, the following improvements have been completed:

- ✅ Flax version of the model for faster inference.
- ✅ Covariate support via XReg (see Oct. 2025 update).
- ✅ Documentation, examples, and agent skill (see `timesfm-forecasting/`).
- ✅ Fine-tuning example with LoRA via HuggingFace Transformers + PEFT (see `timesfm-forecasting/examples/finetuning/`).
- ✅ Unit tests for core layers, configs, and utilities (see `tests/`).

```
# Install the package with torch
pip install timesfm[torch]
# Or with Flax
pip install timesfm[flax]
# And when XReg is needed
pip install timesfm[xreg]
```
- 
Clone the repository: `git clone https://github.com/google-research/timesfm.git cd timesfm`
- 
Create a virtual environment and install dependencies using `uv`:# Create a virtual environment uv venv # Activate the environment source .venv/bin/activate # Install the package in editable mode with torch uv pip install -e .[torch] # Or with flax uv pip install -e .[flax] # And when XReg is needed uv pip install -e .[xreg] 
- 
[Optional] Install your preferred `torch`/`jax`backend based on your OS and accelerators (CPU, GPU, TPU or Apple Silicon).:

- Install PyTorch.
- Install Jax for Flax.

```
import torch
import numpy as np
import timesfm
torch.set_float32_matmul_precision("high")
model = timesfm.TimesFM_2p5_200M_torch.from_pretrained("google/timesfm-2.5-200m-pytorch")
model.compile(
    timesfm.ForecastConfig(
        max_context=1024,
        max_horizon=256,
        normalize_inputs=True,
        use_continuous_quantile_head=True,
        force_flip_invariance=True,
        infer_is_positive=True,
        fix_quantile_crossing=True,
    )
)
point_forecast, quantile_forecast = model.forecast(
    horizon=12,
    inputs=[
        np.linspace(0, 1, 100),
        np.sin(np.linspace(0, 20, 67)),
    ],  # Two dummy inputs
)
point_forecast.shape  # (2, 12)
quantile_forecast.shape  # (2, 12, 10): mean, then 10th to 90th quantiles.
```
