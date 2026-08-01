# Binary Vector Embeddings

Binary quantization turns vector embeddings from 32-bit floats into single bits — 1 if positive, 0 otherwise — then uses Hamming distance (XOR + popcount, a single CPU instruction) instead of cosine similarity. The shock is how little accuracy you lose: 95%+ retrieval accuracy at 32× compression and ~25× speedup, across multiple embedding models none of which were explicitly trained for this.

---

## Key Quotes

> Binary quantization is the process of converting a float32 vector embedding into a binary one. It is literally as simple as checking whether each number is greater than 0. If so, the binary dimension is 1 and if it's less than or equal to 0 it's 0. Does this sound too easy? I thought so too but the results speak for themselves.

The simplicity is the point. This isn't product quantization, scalar quantization, or any of the heavier compression schemes — it's a single sign check per dimension. The fact that this works at all, let alone at 95%+ retention, says something about how much redundancy lives in those float32 dimensions.

> it's not like the authors of this model specifically designed it to support binary quantization. It's just that this model spontaneously developed these properties.

Evan Schwartz's observation here is the most interesting one: binary quantization is an **emergent property** of these embedding models, not a design goal. The models learned representations where the sign of each dimension carries most of the semantic signal, and the magnitude is largely noise. This rhymes with observations in LLM quantization — a lot of those float32 bits aren't doing meaningful work.

> No change of infra needed! I just changed the distance function and switched my float32 vector database to a binary one and it worked.

The infrastructure story is the killer feature. Binary quantization doesn't require new databases, new hardware, or new deployment patterns. Change the distance metric and suddenly your existing vector store holds 32× more vectors and searches 25× faster.

## Key Themes

#vector-search #embeddings #quantization #optimization #similarity

**Binary quantization vs. Matryoshka embeddings.** The empirical comparison is stark: slicing a Matryoshka embedding to 12.5% of its dimensions preserves only 67% of retrieval performance, while binary quantization at 3.125% of the size preserves 96%. They attack different axes — one reduces dimensionality, the other reduces precision per dimension — and the data says precision-per-dimension has far more slack than dimensionality does.

**Combining techniques.** A 512-dimension binary-quantized Matryoshka embedding hits 1.56% of the original size at 90.76% accuracy. For production systems where storage cost dominates, this is the play. At 1.56% size, you're storing ~64× more vectors in the same memory budget, which can be the difference between "keep everything in RAM" and "spill to disk."

**Model-dependent variance.** Not all models binarize equally well. `mxbai-embed-large-v1` retains 96.45% while `nomic-embed-text-v1.5` drops to 87.7%. The 9-point spread matters: if you're picking an embedding model and binary search is in your future, test this property during model selection.

## Critical Analysis

The 95%+ retention claim is real and reproducible, but it's worth being precise about what it measures. The MTEB retrieval benchmark tests whether the correct document appears in the top-k results — it says nothing about ranking quality within those results or about fine-grained similarity distinctions. If your application cares about the *ordering* of results more than binary inclusion/exclusion, the practical degradation may be larger than 4%.

The Hamming distance speedup is real but context-dependent. XOR + popcount on 1024-bit vectors (128 bytes) is blazing fast on CPU. But if you're already running vector search on GPU with batched cosine similarity, the relative advantage shrinks — GPUs are optimized for the float32 path. The 25× mean speedup is a CPU number. For GPU-accelerated vector databases, benchmark your own stack.

The infrastructure simplicity argument — "just change the distance function" — is more true for some vector databases than others. pgvector supports Hamming distance natively. Qdrant supports binary quantization as a first-class feature. But not every vector DB makes binary index types a drop-in replacement, and some require reindexing.

The emergent-property observation is the deepest insight in the post. If model trainers started explicitly optimizing for binary-retrieval retention during training, the 95% number would likely go higher. This is an underexplored axis in embedding model development — most benchmarks still report float32 numbers, creating no pressure to preserve the sign-only signal.

---

*Sources: [[raw/binary-vector-embeddings-are-so-cool]]*
*Last updated: 2026-08-01*
