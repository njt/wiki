---
url: https://emschwartz.me/binary-vector-embeddings-are-so-cool/
title: "Binary vector embeddings are so cool"
author: Evan Schwartz
date_fetched: 2026-08-01
date_published: 2024-11-11
---

Evan Schwartz explains binary quantization for vector embeddings: convert each float32 dimension to a single bit (1 if positive, 0 otherwise). This enables similarity search via Hamming distance — XOR plus popcount, a single CPU instruction — instead of floating-point cosine similarity across thousands of dimensions.

The results are striking. Binary embeddings achieve 95%+ retrieval accuracy with 32× compression and a mean 25× retrieval speedup. On MixedBread's `mxbai-embed-large-v1`, binary embeddings at 3.125% of the original size retain 96.45% of retrieval performance (MTEB score 52.46 vs. 54.39 for float32). Other models tested ranged from 87.7% to 94.6% accuracy retention.

Binary quantization handily beats Matryoshka (dimension-slicing) compression alone: slicing to 12.5% of dimensions drops performance to 67.34%, while binary quantization at 3.125% size keeps 96.45%. Combining both — a 512-dimension binary-quantized Matryoshka embedding — gets down to 1.56% of original size while retaining 90.76% accuracy.

Schwartz applied this to his personal content feed project (Scour) after hitting slow vector lookups. The switch required only a distance-function change — no infrastructure migration. He notes the effect is emergent: the embedding models weren't designed for binary quantization, yet the property appears spontaneously.
