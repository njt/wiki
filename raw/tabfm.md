---
url: https://github.com/google-research/tabfm
title: "TabFM: Tabular Foundation Models"
author: Google Research
date_fetched: 2026-07-03
date_published: 2026-06-29
---

# TabFM — Raw Ingest Analysis

## Source repo

- **URL**: https://github.com/google-research/tabfm
- **License**: Apache 2.0
- **Language**: Python (JAX + PyTorch)
- **Size**: ~3,300 loc for sklearn interface, ~2,800 loc for JAX model, ~460 loc for PyTorch model, ~630 loc for memory-efficient attention
- **Dependencies**: JAX 0.10.1, Flax 0.12.7 (nnx API), PyTorch 2.12.1, einops, scikit-learn, huggingface-hub, numpy, pandas

## Architecture

TabFM is a transformer-based architecture for in-context learning on tabular data. It processes a dataset as a 3D tensor (batch × rows × features) through three sequential stages:

### Stage 1: Cell-wise Embedding (CellEmbedder)

Implemented in `tabfm/src/jax/model.py:1489-1779` (JAX) and `tabfm/src/pytorch/model.py:256-329` (PyTorch).

Each cell value is expanded using Fourier features (sin/cos transformations with learned frequency banks) before being projected through a linear layer:

- **Numerical features**: sin/cos of `value * fourier_frequencies`
- **Categorical features**: sin/cos of `value * fourier_frequencies_cat` (separate frequency bank)

The Fourier expansion uses 32 frequencies (`fourier_features_num_frequencies=32`), producing 64-dimensional feature representations that are then linearly projected to `embed_dim`.

Features are organized into overlapping groups of 3 (`feature_group_size=3`) using shifts of (2^i - 1), creating a sliding-window-like overlapping view:

```
For a feature at position j, the group is: 
[j + 0, j + 1, j + 3]  (when i=0,1,2: 2^0-1=0, 2^1-1=1, 2^2-1=3)
```

For training rows, label embeddings are added to the cell embeddings; test rows get only cell embeddings.

### Stage 2: Column-wise and Row-wise Interaction (ColEmbedding, RowInteraction)

Two rounds of alternation, each round consisting of:

1. **Column attention** (`ColEmbedding`, `tabfm/src/jax/model.py:1785-1950`): A Set Transformer with induced points (`num_inds=128`) that attends across rows within each column independently. Uses `InducedSelfAttentionBlock` which first attends induced points to the full set, then back from induced to elements — the standard Set Transformer pattern from Lee et al. (2019). Tensor is reshaped from `[B, T, HC, E]` to `[B*HC, T, E]` so each column is processed independently.

2. **Row attention** (`RowInteraction`, `tabfm/src/jax/model.py:1957-2055`): A standard transformer encoder with RoPE (rotary positional embeddings, `rope_base=100000`) that attends across features within each row independently. Learnable CLS tokens (`row_num_cls=4`) are prepended before row attention. Tensor is reshaped from `[B, T, HC+CLS, E]` to `[B*T, HC+CLS, E]`.

The second `ColEmbedding` and second `RowInteraction` use the same architecture but the second `RowInteraction` drops the CLS tokens (`output_full=False`), outputting only CLS representations — these become "row representations" for the ICL stage. The ICL dimension is therefore `embed_dim * row_num_cls = 128 * 4 = 512`.

### Stage 3: In-Context Learning (ICLearning)

Implemented in `tabfm/src/jax/model.py:2073-2259`.

A 12-layer decoder-only transformer (`icl_num_blocks=12`) without RoPE that processes row representations from stage 2:

- **Train rows**: label embedding is added to row representation, can attend to all other train rows (causal mask)
- **Test rows**: no label, can attend only to train rows (causal mask blocks attention to test tokens with higher indices)

Label encoding differs by task:
- **Classification**: `OneHotAndLinear(max_classes, d_model)` — one-hot encode the class index, then linear projection
- **Regression**: `MLP(1, [d_model*2], d_model)` — scalar value through a tiny MLP

The decoder is a single MLP:
- **Classification**: `MLP(d_model, [d_model*2], max_classes)` → logits over classes
- **Regression**: `MLP(d_model, [d_model*2], 1)` → scalar prediction

### Attention Implementations

Three attention implementations are available (`tabfm/src/jax/model.py:61-66`):

1. **JAX** (default): Uses `jax.nn.dot_product_attention`
2. **JAX_VMAP_ON_HEAD_DIM**: vmaps `jax.nn.dot_product_attention` over head dimension for memory savings
3. **FLASH**: Custom memory-efficient attention (`tabfm/src/jax/memory_efficient_attention.py`)

The custom attention implementation (`memory_efficient_attention.py:187-296`) processes queries and keys in chunks with online softmax — the same core algorithm as Flash Attention. Key chunk sizes: `query_chunk_size=1024`, `key_chunk_size=2048`. The implementation uses a numerically stable running-sum approach:

```
correction = exp(previous_max - max_so_far)
numerator = numerator * correction + sum(values * corrected_weights)
denominator = denominator * correction + sum(corrected_weights)
```

### Prefill/Decode Pattern

`TabFM.prefill()` (line 2583) processes training data and caches KV states. `TabFM.decode()` (line 2687) uses cached KVs for efficient test inference without re-processing the full training set.

The cache stores per-layer keys and values for the column transformers and the ICL transformer. Training rows are padded to multiples of 128 for Flash Attention efficiency.

### Batch Processing with JAX Sharding

The `TabFMClassifier` and `TabFMRegressor` classes (`classifier_and_regressor.py:2280-2300`) use JAX multi-device sharding:
- Each ensemble member is placed on a different TPU/GPU device
- `jax.pmap` is used for data-parallel inference
- `multihost_utils` handles cross-host synchronization

### Per-Dimension Scale (PerDimScale)

`tabfm/src/jax/model.py:169-194` implements learned per-dimension scaling for attention queries:

```python
x * (1.442695041 / sqrt(num_dims) * softplus(per_dim_scale))
```

The constant `1.442695041 ≈ 1/ln(2)` converts the natural-log softplus to a base-2 scale. This replaces the fixed `1/sqrt(d_k)` scaling used in standard attention — the model learns which attention dimensions to emphasize or suppress.

Notably, the T5 convention of folding the `1/sqrt(depth)` into weight initializers (rather than in the attention computation itself) is noted in the code and is the reason `rescale_logits` defaults to `False` (line 462-467 of memory_efficient_attention.py).

## Ensemble Generation

The `EnsembleGenerator` class (`classifier_and_regressor.py:1017-1780`) creates diverse views of the same dataset:

1. **Multiple normalization methods**: `["none", "power"]` by default — raw scaling vs PowerTransformer
2. **Feature shuffling**: Random permutation of feature columns per ensemble member, with `max_num_features=500` subsampling
3. **Class-label shifts** (classification only): Random shift of class labels to prevent overfitting to specific label positions
4. **Categorical value permutations**: Randomly swap categorical values per ensemble member (controlled by `permute_categorical=True`)
5. **Feature crosses**: Pairwise feature products, with "split" allocation — even-indexed members get none, odd-indexed get sqrt(n_features) crosses
6. **SVD structural features**: TruncatedSVD components added with the same split allocation pattern
7. **Row subsampling**: Subset of training rows per ensemble member (controlled by `max_num_rows`)

Default ensemble size from `TabFMClassifier.ensemble()`: 32 members, with feature crosses and SVD enabled, NNLS blending, and Platt/vector calibration.

## Preprocessing Pipeline

`TransformToNumerical` (`classifier_and_regressor.py:366-530`) handles mixed data types:

- **Categorical**: `CategoricalOrdinalEncoder` with alphabetical ordering (matching sklearn convention), min_frequency filtering
- **Datetime**: `DatetimeTransformer` extracts year/month/day/dayofweek + Unix nanoseconds
- **Numerical**: RobustScaler → StandardScaler → QuantileTransformer chain
- **Outlier removal**: Z-score based pruning (threshold default 4.0)

A deliberate design choice: datetime detection uses a `random_state=0` seed that is explicitly separate from the ensemble seed `_DEFAULT_RANDOM_STATE=42`. The comment at line 90 explains: "column-type detection must stay stable across ensemble seeds so the feature schema doesn't change when only the model seed varies."

## Calibration and Blending

### NNLS Blending

`scipy.optimize.nnls` finds optimal non-negative weights for ensemble members from out-of-fold predictions. A beta parameter (default 0.5) controls regularization — blending toward uniform weights. Classification applies NNLS in log-probability space.

### Platt Scaling / Vector Scaling

- **Binary**: Platt scaling fits a sigmoid calibration curve
- **Multiclass**: Vector scaling fits a diagonal+intercept per-class calibration

Both use out-of-fold predictions (5-fold CV) to avoid overfitting. The `min_rows_for_single_val_split` parameter (default 2) handles small datasets — if any validation fold would be smaller, CV is skipped.

### Temperature-scaled softmax

Default `softmax_temperature=0.9` produces sharper predictions than standard softmax. Applied at the ensemble-averaging stage, not per-member.

## Key Design Decisions

### Dual Backend Strategy

JAX for training and high-performance TPU inference; PyTorch for HuggingFace distribution and CPU/GPU compatibility. The PyTorch port (`tabfm/src/pytorch/model.py`) is described as "EXPERIMENTAL faithful-ish" but parity-verified to ~1e-6 in float32. The structure matches JAX 1:1 — same module names, same parameter layout — enabling mechanical weight conversion via `tabfm/src/hugging_face/torch_convert.py`.

### sklearn API Compliance

`TabFMClassifier` extends `ClassifierMixin` and `TabFMRegressor` extends `RegressorMixin`. They implement the standard `fit()`/`predict()`/`predict_proba()` interface plus `get_params()`/`set_params()`. Label encoding uses alphabetical ordering to match sklearn's `LabelEncoder` convention.

### Chunked Processing Throughout

Every stage supports chunking over independent dimensions to avoid materializing huge intermediates:
- `CellEmbedder.row_chunk_size` — chunks the Fourier expansion over rows
- `ColEmbedding.col_chunk_size` — chunks the independent column axis
- `RowInteraction.row_chunk_size` — chunks the independent row axis
- `MultiheadAttentionBlock.ffn_chunk_size` — chunks the FFN over tokens in PyTorch

### Zero-shot Inference

The model never trains on the user's dataset. The `fit()` method only fits the preprocessing pipeline and ensemble generator — no gradient updates. All learning happens through in-context attention at inference time.

## Comparison Points

- Unlike XGBoost/CatBoost (tree-based), TabFM uses in-context learning rather than gradient boosting
- Unlike TabPFN (Prior-Data Fitted Networks for tabular data), TabFM's ICL transformer is a decoder-only architecture with prefill/decode, not a Perceiver or encoder-only model
- Unlike standard transformer tabular approaches (FT-Transformer, TabTransformer), TabFM does not require training on the target dataset at all — it's a true foundation model for tables
- The ensemble strategy (feature crosses, SVD, class shifts) is reminiscent of scikit-learn stacking but done at the data-augmentation level rather than model level
- The Set Transformer for column attention is a more principled column-permutation-invariant approach than standard positional embeddings
