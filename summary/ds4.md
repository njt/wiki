---
url: https://github.com/antirez/ds4
title: "DwarfStar 4 (DS4) — DeepSeek V4 Flash Inference Engine"
author: antirez (Salvatore Sanfilippo), with GPT 5.5 assistance
date_fetched: 2026-05-18
date_published: 2026-05-14
tags: [inference, c, metal, cuda, deepseek, gguf, quantization, local-ai]
topics:
  - misc
---

## Source Analysis

A complete walkthrough of the ds4.c codebase at commit fetched 2026-05-18. The repo is ~65K lines of C, Objective-C, Metal, and CUDA code implementing a single-model inference engine for DeepSeek V4 Flash (284B parameter MoE model).

### File Tree (source files only)

```
ds4.c          18,403 lines — core engine: GGUF loading, CPU reference, Metal graph driver, tokenizer, session, KV cache
ds4.h             200 lines — public API boundary for CLI/server
ds4_metal.m    14,721 lines — Objective-C Metal runtime, kernel wrappers, buffer management
ds4_cuda.cu    10,723 lines — CUDA backend (DGX Spark / generic)
ds4_server.c   15,581 lines — HTTP server: OpenAI/Anthropic/Responses API, streaming, tool calls
ds4_eval.c      3,339 lines — built-in benchmark: GPQA+AIME+SuperGPQA+COMPSEC
ds4_cli.c       1,379 lines — interactive REPL with linenoise
ds4_bench.c        ??? lines — throughput benchmark with frontier sampling
ds4_gpu.h         810 lines — GPU tensor/kernel interface (shared by Metal and CUDA)
rax.c / rax.h    ~2,000 lines — radix tree (from Redis)
linenoise.c        ~800 lines — line editing library (from Redis)
metal/*.metal     17 files — Metal compute kernels (attention, MoE, norms, RoPE, compressor, etc.)
tests/             unit tests, logprob vectors, long-context smoke tests
gguf-tools/        quantization tools, imatrix collection, quality scoring
dir-steering/      directional activation steering tools
speed-bench/       CSV benchmarks and SVG plots
```

### Architecture Deep Dive

**Fixed Model Shape**: The engine hardcodes the exact DeepSeek V4 Flash architecture as compile-time constants at the top of ds4.c: 43 layers, 4096 embedding dim, 129,280 vocab, 64 attention heads (but only 1 KV head — multi-query attention), 512-dim head, 64-dim RoPE, 256 routed experts (6 active per token + 1 shared), 2048-dim FFN experts, 128-token sliding window attention, 4-stream hyper-connections (HC), 3 hash-layer routing layers.

**Layer Compression**: Layers 0-1 have no compression (raw SWA only). Even-numbered layers from 2 upward use ratio-4 compression with a learned indexer that selects the top-512 compressed rows. Odd-numbered layers use ratio-128 compression without an indexer. The compressor maintains rolling KV/score state and appends pooled rows at ratio boundaries.

**GPU Backend Architecture**: The `ds4_gpu.h` interface defines ~60+ kernel operations. The C engine calls these through function pointers; the Metal and CUDA backends implement them independently. The interface includes: embeddings, matmuls (Q8_0, F16, F32 variants), RMS norm, RoPE tail rotation, DS4-specific FP8 KV quantization, compressor update/prefill/replay, attention (raw heads, mixed heads, indexed mixed heads, flash attention prefill), router selection (top-k/hash), routed MoE (IQ2_XXS, Q2_K, Q4_K), hyper-connection split/expand with Sinkhorn iterations, directional steering projection, and indexer scoring/topk.

**Whole-Model Graph Execution**: On Metal, the engine builds one complete command buffer chain per layer. For prefill, tokens are batched into 2048-token chunks (configurable via `DS4_METAL_PREFILL_CHUNK`). For decode, single tokens are processed sequentially. The graph schedule inside `metal_graph_encode_layer_batch()` calls: HC split+weighted sum+norm → Q/KV projections with fused RMS norm → RoPE tail → FP8 KV quantize → store raw KV → attention (choosing among 5 different attention kernel paths depending on zero-prefix vs. nonzero-chunk, ratio, and batch size) → attention output (low-rank Q8) → HC expand → FFN RMS norm → router (hash for layer 3, top-6 for others) → shared SwiGLU + routed MoE → shared down projection with HC expand → final HC expand.

**mmap Loading**: The GGUF file is mapped with `mmap(NULL, st.st_size, PROT_READ, MAP_SHARED, fd, 0)`. Tensor data stays in the kernel page cache until accessed. On Metal, slices of the mapping are wrapped as MTLBuffer views via `MTLBuffer initWithBytesNoCopy:length:options:deallocator:` — zero-copy. The engine caches frequently-used model ranges in Metal residency sets.

**KV Cache Design**: Per-layer state:
- `raw_kv`: Ring buffer of 128-token sliding window attention rows (F32 on CPU, F16 on GPU)
- `attn_comp_kv`: Compressed KV rows (pooled at ratio boundaries, e.g., every 4 or 128 tokens)
- `attn_state_kv` / `attn_state_score`: Rolling compressor state (ratio-4 layers get learned states)
- `index_comp_kv` / `index_state_kv` / `index_state_score`: Separate indexer compressor for ratio-4 layers
- `n_raw`, `n_comp`, `n_index_comp`: Live row counts

**Disk KV Cache**: Files named `<SHA1(rendered_prefix)>.kv`, format: KVC header (48 bytes, little-endian) → rendered text → DS4 session payload (checkpoint tokens, logits, per-layer compressed row counts, raw SWA rows in logical order, compressed rows, compressor/indexer frontiers) → optional tool ID map (KTM section). Save moments: cold (after first prompt), continued (every ~10K tokens), evict (before replacing live session), shutdown.

**Tool Call Handling**: DeepSeek V4 emits tool calls as DSML text. The server captures exact sampled bytes keyed by unguessable API tool IDs (radix tree). On the next turn, if the client sends that tool ID back, the exact DSML bytes are replayed into the prompt — ensuring the KV cache prefix matches byte-for-byte. Fallback: canonical JSON-to-DSML rendering. During generation, DSML syntax tokens are forced to temperature=0 (argmax), but argument payloads (code, file contents, string values) use the request's normal sampling.

**CPU Decode as Reference**: The CPU path pre-allocates all scratch buffers once per session (never allocates during generation). Uses `xmalloc+memset` instead of `calloc` to avoid lazy zero-page VM faults that can crash macOS under large model mappings. A persistent thread pool (up to 32 threads) handles row-parallel operations with a barrier dispatch pattern. ARM NEON SIMD with optional dot-product instructions for IQ2 quantized dot products.

**Asymmetric Quantization**: Routed experts use IQ2_XXS for gate/up and Q2_K for down (or Q4_K in the high-memory variant). Shared experts, attention projections, router, and embeddings stay at full precision (F16). This is the key innovation that makes 284B parameters fit in ~96GB RAM with usable quality.

**Hyper-Connection Primitives**: Each layer passes activations as 4×4096-dim streams (HC). Before a sublayer (attention or FFN), the 4 streams are concatenated, passed through a Sinkhorn-normalized mixer (learned scale + base weights, 20 Sinkhorn iterations), and reduced to 4096. After the sublayer, the output is expanded back into 4 streams using learned post/comb weights. RMS norm is applied throughout with epsilon 1e-6.

**Server Architecture**: The HTTP server uses raw sockets with poll, one worker thread per client for request parsing, but inference is serialized through a single graph worker that owns the mutable session. This avoids KV cache race conditions. The server speaks three wire protocols: OpenAI `/v1/chat/completions`, Anthropic `/v1/messages`, and OpenAI Responses `/v1/responses` (for Codex CLI). SSE streaming, CORS, tool calling with streaming tool-call argument deltas.

### Dependencies

- **Zero external dependencies beyond system libraries**: libm, pthreads, Foundation.framework, Metal.framework (macOS); libcudart, libcublas (CUDA)
- Uses **linenoise** (bundled, from Redis) for CLI line editing
- Uses **rax** (bundled, from Redis) for radix tree in the server's tool-call map
- Build system: plain GNU Make with platform detection
- C99 standard, no C++

### What Makes This Project Special

1. It's a bet that one model, one engine, one integration is better than a generic runner supporting hundreds of models. The value is in the deep integration, not in model flexibility.
2. The disk KV cache is treated as a first-class citizen, not a bolt-on. Careful alignment to chunk boundaries, conservative trimming, and SHA1-keyed by rendered prefix bytes.
3. The 2-bit quant recipe works well enough for coding agents because it only quantizes routed experts (which are 48/49 of the parameters in DeepSeek V4).
4. The project was built in ~1 week with GPT 5.5 assistance, but the quality is remarkably high — comprehensive test vectors, official logprob comparison, long-context tests, production-grade Metal kernel implementations.
5. The engineering priority order is explicit: correctness before speed, Metal before CUDA, disk KV cache before everything else.

### Code Quality Notes

- The code is deliberately "vertical" — the main file ds4.c is 18K lines and owns nearly everything rather than splitting into many modules. This is antirez's signature style (cf. Redis).
- Comments are sparse but placed exactly where the model mechanics, cache lifetime, or memory policy are non-obvious.
- The CPU path has an unusual `calloc`→`malloc+memset` substitution with an explicit comment explaining it prevents macOS kernel panics from VM first-touch faults.
- The engine uses `DS4_NO_GPU` compile flag to produce a CPU-only build (ds4_cpu.o) with the same source file.
