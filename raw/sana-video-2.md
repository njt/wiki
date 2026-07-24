---
url: https://nvlabs.github.io/Sana/Video2/
title: "SANA-Video 2.0: Hybrid Linear Attention with Attention Residuals for Efficient Video Generation"
authors: Junsong Chen, Jincheng Yu, Yitong Li, Shuchen Xue, Haozhe Liu, Jingyu Xin, Yuyang Zhao, Tian Ye, Zhangjie Wu, Zian Wang, Daquan Zhou, Ping Luo, Song Han, Enze Xie
date_fetched: 2026-07-25
date_published: 2026
affiliation: NVIDIA Research (Efficient AI Team & Singapore Lab)
arxiv: 2607.21553
doi: 10.48550/arXiv.2607.21553
---

# SANA-Video 2.0 — Hybrid Linear Attention with Attention Residuals for Efficient Video Generation

## Authors
Junsong Chen, Jincheng Yu, Yitong Li, Shuchen Xue, Haozhe Liu, Jingyu Xin, Yuyang Zhao, Tian Ye, Zhangjie Wu, Zian Wang, Daquan Zhou, Ping Luo, Song Han, Enze Xie

**Affiliation:** NVIDIA Research (Efficient AI Team & Singapore Lab)

## Publication Details
- eprint: 2607.21553 (arXiv, primaryClass cs.CV)
- Year: 2026
- DOI: 10.48550/arXiv.2607.21553

## Links
- [Paper on arXiv](https://arxiv.org/abs/2607.21553)
- [Code on GitHub](https://github.com/NVlabs/Sana)

## Key Metrics (callout banner)
- **84.30** VBench Total
- **13.06s** for 720p/5s on one H100
- **3.2× faster** than softmax at 60s
- **120× faster** than Wan 2.2 14B

## One-H100 Latency Benchmarks (720p / 5s / 40 steps)

| Model | Latency (seconds) |
|---|---|
| Wan 2.2 14B | 1556 |
| Hunyuan | 788 |
| LTX-2.3 | 130 |
| Ours 14B | 69.3 |
| **Ours 5B + Sol** | **13.06** |

*(Note: log scale, lower is better)*

## Abstract & Technical Details

The paper introduces a hybrid video diffusion transformer at two scales (5B and 14B) under a unified architecture, targeting high-quality 720p video generation on a single GPU. The system matches full-softmax video DiTs in quality while benefiting from linear attention's favorable long-sequence scaling.

### Hybrid Linear-Softmax Attention
The method "combines gated linear attention for O(N)-dominated mixing with periodic gated-softmax anchors at a 3:1 ratio." This restores full-rank token interactions that pure linear attention lacks, avoiding quadratic attention throughout.

### Block Attention Residuals (AttnRes)
Completed block summaries are routed into later linear layers, enabling anchor-feature reuse. This boosts deep-layer effective rank by approximately 12%.

### Training Approach
The model is trained from scratch — the complete hybrid is learned directly rather than linearizing pretrained models. Reduced-resolution proxy studies identified 25% softmax as the optimal quality-efficiency trade-off.

### Performance Highlights
- At 40-step sampling, SANA-Video 2.0 achieves a VBench score of 84.30 in 13.2 seconds at 480p on a single H100, remaining competitive with far larger softmax video DiTs.
- The compiled DiT forward pass is "3.2× faster than a matched full-softmax baseline at 720p/60s," with the gap widening as video duration increases.
- Full-stack **Sol-Engine optimization** (kernel fusion, caching, sparse attention) accelerates the backbone by an additional **3.58×**, bringing the 5B pipeline to 13.06s at 720p/5s.
- Overall, the system is **120× faster than Wan 2.2-A14B** on one H100.

## Bottom-Line Claim
The authors state their "hybrid design recovers softmax-level expressiveness at substantially reduced cost, unlocking scalable long, high resolution video generation."

## Additional Notes
- The page notes "VIDEO SOURCE COMING SOON" for a video showreel section.
- BibTeX citation is provided on the page.