---
url: https://emschwartz.me/binary-vector-embeddings-are-so-cool/
title: "Binary vector embeddings are so cool"
author: Evan Schwartz
date_fetched: 2026-08-01
date_published: 2024-11-11
---

# Binary vector embeddings are so cool

Binary vector embeddings are a technique that transforms standard float32 vector embeddings into binary representations. Each dimension becomes a single bit: 1 if the float value is positive, 0 otherwise. This enables similarity search via Hamming distance (XOR + popcount — a single CPU instruction) instead of cosine similarity (floating-point dot products across thousands of dimensions).

The results:

- **95+% retrieval accuracy with 32× compression and ~25× retrieval speedup**
- For MixedBread's `mxbai-embed-large-v1`: binary embeddings are 3.125% of the default size yet retain 96.45% of retrieval performance (MTEB score 52.46 vs. 54.39 for float32)
- Other models: all-MiniLM-L6-v2 (93.79%), nomic-embed-text-v1.5 (87.7%), cohere-embed-english-v3.0 (94.6%)

Matryoshka embeddings (slicing dimensions) degrade much faster: cutting to 12.5% of size drops performance to 67.34%, whereas binary quantization at 3.125% size keeps 96.45%.

Combining both techniques: a 512-dimension binary-quantized Matryoshka embedding is 1.56% of the original size and retains 90.76% accuracy.

Retrieval speedup from Hamming distance: 15×-45× speedup with a mean of 25×.

The author applied binary quantization to his personalized content feed project, Scour, after noticing slow vector lookups. The speedup was enough that he made "No change of infra needed!"

> Binary vector embeddings are a really simple, cool trick: they let you transform regular (float32) vector embeddings into binary vectors. In other words, each dimension of the vector is represented as a single bit (0 or 1) instead of a 32-bit floating point number. This means you can store 32x as many embeddings in the same amount of memory and you can use Hamming distance instead of cosine similarity to compare them, which is a 15-45x speedup (mean of 25x). The craziest part is that binary vector embeddings still retain ~95% of the retrieval accuracy!

> Binary quantization is the process of converting a float32 vector embedding into a binary one. It is literally as simple as checking whether each number is greater than 0. If so, the binary dimension is 1 and if it's less than or equal to 0 it's 0. Does this sound too easy? I thought so too but the results speak for themselves.

> I just think this is pretty mind-blowing. ...it's not like the authors of this model specifically designed it to support binary quantization. It's just that this model spontaneously developed these properties.

> No change of infra needed! I just changed the distance function and switched my float32 vector database to a binary one and it worked.

The author plans to follow MixedBread's future work on this topic.
