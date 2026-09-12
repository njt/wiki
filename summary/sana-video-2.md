---
topics:
  - ai-research-and-models
---
# SANA-Video 2.0 (Summary)

NVIDIA Research introduces a hybrid video diffusion transformer at 5B and 14B scales that mixes gated linear attention (3/4 of layers) with periodic gated-softmax anchor layers (1/4) to get softmax-level quality at linear-attention speeds. Block Attention Residuals route completed block summaries into later layers for a ~12% effective-rank boost. Trained from scratch rather than linearizing pretrained models. On a single H100: 84.30 VBench, 13.06s for 720p/5s with the 5B model + Sol-Engine optimizations — 120× faster than Wan 2.2 14B. The key architectural insight: 25% softmax layers turns out to be the optimal quality-efficiency trade-off, restoring the full-rank token interactions that pure linear attention loses.

*Source: [[raw/sana-video-2]]*
*Date: 2026-07-25*
