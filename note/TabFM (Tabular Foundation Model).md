# TabFM (Tabular Foundation Model)

TabFM is Google Research's foundation model for tabular data — a transformer that predicts labels for new rows by reading (features, labels) pairs as context, without ever training on your dataset. Think of it as a GPT for spreadsheets: you give it example rows with known outcomes, and it infers the pattern to classify or regress over new rows.

It's sklearn-compatible (`TabFMClassifier` / `TabFMRegressor`), ships with pre-trained weights on HuggingFace, and supports both JAX (TPU training) and PyTorch (inference) backends. Released June 2026 under Apache 2.0.

## Architecture

The model processes tables as 3D tensors (batch × rows × features) through three stages:

**Stage 1: Cell Embedding.** Each cell value is expanded through sin/cos Fourier features with 32 learned frequencies, then projected to embedding space. Numerical and categorical cells use *separate* frequency banks — the model learns different representations for continuous vs. discrete values. Features are grouped in overlapping triplets with binary-offset shifts (2^i − 1), giving the model a local context window over adjacent columns.

**Stage 2: Column/Row Processing.** Two rounds of alternation between column-wise and row-wise attention. Column attention uses a Set Transformer with 128 induced points — attending across rows independently per-column, which is column-permutation-invariant by design. Row attention uses a standard Transformer with RoPE rotary embeddings and prepended [CLS] tokens, attending across features per-row.

**Stage 3: In-Context Learning.** A 12-layer decoder-only Transformer processes row representations from stage 2. Train rows get label embeddings added and can attend to each other. Test rows see only the train context (causal mask). For classification, labels are one-hot encoded + linearly projected; for regression, a 6-hidden-unit MLP encodes the scalar target.

The key architectural insight: the model uses a **prefill/decode pattern** — training rows are processed once with KV caching, then test rows are decoded in a second pass without recomputing the full training context. This is the same pattern used in LLM serving.

## Key Techniques

**Fourier feature cell embeddings.** Instead of directly embedding raw values, TabFM expands each cell through sin/cos with learned frequencies before linear projection. This is a continuous generalization of positional encoding — any scalar value gets mapped to a rich high-dimensional representation. The `CellEmbedder` uses ~600 lines in the JAX model (`tabfm/src/jax/model.py:1489-1779`).

**Memory-efficient chunked attention.** The code has a custom Flash Attention implementation (`tabfm/src/jax/memory_efficient_attention.py`, 628 lines) that processes queries and keys in chunks with online softmax — the same numerically stable running-sum algorithm as the published Flash Attention paper. Chunk sizes default to 1024 (query) and 2048 (keys).

**Per-dimension scaling.** Instead of the universal `1/sqrt(d_k)` attention scaling, `PerDimScale` (`model.py:169-194`) learns a per-dimension scale via softplus: `x * 1.4427 / sqrt(d) * softplus(per_dim_scale)`. The constant 1.4427 ≈ 1/ln(2) converts from natural log to base-2. This lets the model learn which attention dimensions matter and which to suppress.

**Ensemble with data augmentation, not model diversity.** The 32-member ensemble doesn't train 32 different models — it creates 32 different *views* of the same data through feature shuffling, class-label shifts, categorical value permutation, multiplicative feature crosses, and SVD structural features. Even-indexed ensemble members get no augmented features; odd-indexed get sqrt(n_features) crosses and SVD components. Blending weights are learned via non-negative least squares from out-of-fold predictions.

**Every axis is chunkable.** Row, column, and FFN dimensions all support chunked processing to avoid materializing huge intermediate tensors. This is how it handles datasets with thousands of features and rows on limited memory.

## Design Decisions

**Zero-shot, not fine-tuned.** The `fit()` method only fits the preprocessing pipeline — no gradient updates run on your data. All learning happens through in-context attention at inference. This means no GPU is required in the training loop, and the same model works across datasets with completely different schemas.

**Dual backend, same architecture.** JAX for TPU training/production, PyTorch for CPU/GPU inference and HuggingFace distribution. The PyTorch port mirrors the JAX code 1:1 — same module names, same parameter layout — enabling mechanical weight conversion. Parity-verified to ~1e-6 in float32 across all components.

**sklearn convention, no surprises.** Label encoding is alphabetical (matches sklearn's `LabelEncoder`), the API is `fit()`/`predict()`/`predict_proba()`, and the default random seed for type detection is 0 (separate from the ensemble seed 42) so column schemas stay stable when you vary the model seed.

**Aggressive preprocessing.** Before the model ever sees data, TabFM applies: outlier removal (z-score > 4), RobustScaler → StandardScaler → QuantileTransformer, categorical ordinal encoding with min-frequency filtering, datetime expansion (year/month/day/dayofweek + Unix nanos), and rare-category collapsing.

## Comparison Notes

TabFM sits in a small but growing category of tabular foundation models:

- **vs. XGBoost/CatBoost**: Tree-based methods require training on each dataset. TabFM makes predictions from context alone — no gradient boosting, no hyperparameter tuning per dataset. The trade-off: trees can handle arbitrary feature interactions natively; TabFM relies on ensemble augmentation (feature crosses) to capture them.

- **vs. TabPFN** (Prior-Data Fitted Networks): TabPFN uses a Perceiver-style encoder trained on synthetic data. TabFM uses a decoder-only transformer with prefill/decode and does not generate synthetic training data — it's trained directly on real tabular datasets.

- **vs. FT-Transformer / TabTransformer**: These are architectures you *train* on your data. TabFM is a pre-trained model you load and use — the ICL approach means zero dataset-specific training.

- **vs. LLMs for tables**: Some approaches serialize tables as text and feed them to language models. TabFM processes tabular data natively as tensors, with column-permutation-invariant Set Transformer attention and Fourier-embedded cell values — avoiding the tokenization and serialization overhead of text-based approaches. Garnelo & Czarnecki (2026) supply the causal account for *why* this matters: [[Why LLMs Fail at Tabular Prediction]] shows that generic LLM in-context classification capability collapses with dimensionality, and that the failure is not fixable by prompt engineering — the serialisation format, numeric precision, and test-batch size are all red herrings. The dimensionality collapse is the mechanism behind the premise that tabular foundation models exist to solve.

## Tags

#model #foundation-model #tabular-data #transformer #icl #google-research #sklearn #jax #pytorch

## Related

- [[Agent Memory and Context]] — TabFM's in-context approach to tabular data parallels how agents use context for few-shot learning

*Source: [google-research/tabfm](https://github.com/google-research/tabfm) — ingested 2026-07-03*
