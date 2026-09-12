# DS4 (DwarfStar 4)

antirez's single-model inference engine for DeepSeek V4 Flash, written in C with Metal and CUDA backends. ~65K lines of code, zero external dependencies beyond system frameworks, and a deliberate bet that deep integration with one model beats a generic runner supporting hundreds. The project exists thanks to llama.cpp and GGML — it does not link against them but builds on their quantization formats, kernel designs, and ecosystem. Released May 2026, built in ~1 week with heavy GPT 5.5 assistance. The significance: this is the engine that made antirez switch from Claude/GPT to local AI for serious work.

---

## Architecture

The engine is a **monolith with platform backends** — not a library, not a framework. The 18K-line `ds4.c` is the core: it owns GGUF loading, the fixed model shape, CPU reference kernels, Metal graph scheduling, the tokenizer, session lifecycle, and KV cache management. Platform backends (`ds4_metal.m` for Metal, `ds4_cuda.cu` for CUDA) implement the GPU tensor/kernel interface defined in `ds4_gpu.h`.

**Key structural decisions:**

- **Fixed model shape** (`ds4.c:86-109`): 43 layers, 256 routed experts (6 active + 1 shared), 129K vocab, 4096 embedding dim, 64 attention heads with multi-query (1 KV head), 128-token sliding window. These are compile-time `enum` constants, not configuration. If a GGUF doesn't match, the engine fails early.
- **mmap-first loading** (`ds4.c:1200+`): Model weights stay in the kernel page cache. Metal wraps slices as no-copy MTLBuffers. The engine never eagerly copies the full GGUF into user space.
- **Whole-model graph execution** (`ds4.c:12758+`): On Metal, one command buffer per layer with all kernels encoded back-to-back — no GPU/CPU round trips between kernels. Prefill processes tokens in configurable chunks (default 2048).
- **Single mutable session** (`ds4.c:15715`): The session owns the live KV cache and logits. The HTTP server serializes all inference through one worker thread that holds this session — concurrent requests queue up rather than sharing graph state.
- **Proto-handler architecture**: `ds4_server.c` speaks three wire protocols (OpenAI chat completions, Anthropic messages, OpenAI Responses) from a single inference backend, but each protocol handler renders the prompt differently while sharing the same session and KV cache.

---

## Key Techniques

**Asymmetric quantization** (`ds4.c:120-161`, `gguf-tools/`): Only the routed experts are quantized — IQ2_XXS for gate/up projections, Q2_K for down projections. The shared expert, attention weights, router, and embeddings stay at full F16 precision. This exploits DeepSeek V4's architecture: the routed experts are 48/49 of the parameters but contribute only a fraction of the per-token logits. Quantizing them aggressively while keeping the rest pristine is what makes the model fit in ~96GB RAM with usable quality. This is the critical design trade-off — contrast with naive uniform quantization that distributes distortion evenly.

**KV cache with disk as first-class citizen** (`ds4.c:6154-6175`, `ds4_server.c:disk cache`): The KV cache lives in three tiers: raw sliding-window (128 tokens per layer, F16), compressed attention rows (pooled at ratio boundaries — ratio 4 or 128 depending on the layer), and on-disk serialized snapshots. Disk cache files are named `<SHA1(rendered_prefix)>.kv` and contain the complete session payload (checkpoint tokens, logits, per-layer compressed row counts, raw SWA rows, compressor/indexer frontiers, optional tool ID map). The key insight: comparing rendered prompt bytes rather than token IDs for cache lookup avoids BPE boundary mismatches when a client resends a conversation with slightly different tokenization.

**Tool call exact replay** (`ds4_server.c:tool handling`): DeepSeek V4 emits tool calls as DSML text. Agent clients send normalized JSON back. If the server re-rendered the JSON to DSML slightly differently, the KV cache prefix would break. Solution: every tool call gets an unguessable API tool ID, and the sampled DSML bytes are stored in a radix tree (`rax`). On the next turn, if the client sends that tool ID back, the exact original bytes are replayed. The radix map also serializes into the disk KV cache so replays survive server restarts. During generation, DSML syntax tokens (tags, headers, JSON punctuation) are forced to temperature=0; argument payloads (code, file contents) use normal sampling.

**Hyper-connections with Sinkhorn normalization** (`ds4.c:DS4_N_HC=4`, `ds4_gpu.h:hc_split_sinkhorn`): Each layer passes activations as four 4096-dim streams through a learned mixer with 20 Sinkhorn iterations before being reduced to a single 4096-dim row for the sublayer, then expanded back to four streams afterward. The entire CPU decode scratch allocation builds around this 4×4096 layout — about half the scratch memory is HC-dimensioned.

**Persistent decode scratch with anti-calloc** (`ds4.c:6207-6280`): The CPU path pre-allocates ~60 temporary buffers once per session and reuses them for every token. Unusually, it uses `xmalloc+memset` instead of `calloc` because lazy zero-page faults from calloc's shared zero-page, when combined with the huge model mmap, can trigger macOS kernel panics in VM accounting. The comment at `ds4.c:497-508` is worth reading: "We have observed this end in a kernel cpt_mapcnt_inc overflow panic instead of a user-space error."

**KV attention compression** (`ds4.c:411-415`): Layers 0-1 use no compression (raw SWA only). Even layers from 2 onward compress at ratio 4 with a learned indexer that selects top-512 compressed rows. Odd layers compress at ratio 128 without an indexer. The compressor maintains rolling score/KV state and appends pooled rows at ratio boundaries. Prefill processes these on the fly; decode updates the compressor state one token at a time.

---

## Design Decisions

**Optimized for: correctness, then speed.** The AGENT.md says "Preserve correctness before speed. Do not keep a faster path with unexplained attention, KV cache, or logits drift." The test infrastructure reflects this: official logprob vectors from the DeepSeek API serve as the ground truth, and any kernel change must pass logprob comparison before being accepted.

**Optimized for: single model, not generic.** This is the project's strongest opinion. llama.cpp supports hundreds of model architectures; ds4.c supports exactly one. The benefit is code that doesn't have abstraction layers, conditional backends, or indirection. The cost is zero portability to other models. This is the Apple philosophy applied to inference engines.

**Optimized for: MacBook Pro / Mac Studio with 96-128GB RAM.** The 2-bit asymmetric quant is explicitly designed for this hardware class. If you have 256GB+, use the 4-bit variant. If you have less than 96GB, this project isn't for you — yet.

**Sacrificed: batch inference, multi-user concurrency.** The server processes one request at a time through a single graph worker. No batching of independent requests. This keeps the KV cache management simple (one live state, one disk checkpoint) at the cost of throughput. For a personal coding agent use case, this is the right trade-off.

**Sacrificed: portability across model versions.** The fixed model shape means any future DeepSeek V4 Flash variants with different layer counts, dimensions, or expert configurations will require code changes — not just a new GGUF file.

**Innovation: disk KV as first-class citizen.** Most inference engines treat KV cache as pure RAM state. ds4.c treats it as a three-tier hierarchy (raw SWA → compressed RAM → disk snapshots) with careful alignment policies. The cold save trims 32 tail tokens and aligns to 2048-token chunk boundaries to maximize future cache hit probability. Continued saves write at alignment frontiers, typically every ~10K tokens. This is the feature that makes local agent sessions practical — after the first expensive prefill of a 25K-token Claude Code prompt, subsequent turns reuse the cached prefix.

---

## Comparison Notes

**vs. llama.cpp**: llama.cpp is a generic GGUF runner supporting hundreds of model architectures, backends (CPU, CUDA, Metal, Vulkan, SYCL, etc.), and quantization formats. ds4.c is a single-model engine that hardcodes one architecture and two GPU backends. llama.cpp is the ecosystem; ds4.c is the specialized tool. ds4.c exists *because of* llama.cpp — it uses GGUF, IQ2_XXS, Q2_K, Q4_K quantization layouts derived from llama.cpp's work, and acknowledges llama.cpp and GGML as essential prior art. But ds4.c deliberately rejects the "support everything" approach to gain simplicity and model-specific optimizations (compressed KV cache, exact DSML replay, asymmetric quantization policy).

**vs. Ollama / LM Studio**: These are model launchers — they wrap inference engines behind an API and manage model downloading, switching, and serving. ds4.c is the engine itself, with a server built in. You don't swap models; you build the binary for one model and it serves that model. The trade-off is exactly "pick one model, integrate deeply" vs. "support many models, integrate thinly."

**vs. maclocal-api**: [[maclocal-api]] aggregates multiple models behind an OpenAI-compatible API using Apple's MLX and CoreML frameworks. ds4.c is lower-level — it writes its own Metal kernels, manages its own KV cache, and targets exactly one model. maclocal-api is about convenience; ds4.c is about maximum performance for one configuration.

**vs. vLLM / TensorRT-LLM**: These are production serving frameworks optimized for throughput, batching, and multi-user concurrency. ds4.c is optimized for a single user on a single machine. The server batches nothing; the disk KV cache is designed for session resumption, not request coalescing.

**Related in the wiki**: [[A Few Words on DS4]] (the launch post), [[Local and Open Source Inference]], [[maclocal-api]], [[Self-Hosted LLMs]].

---

## Tags

#tool #project #inference #local-ai #c #metal #cuda #open-source

---

*Sources: [[summary/ds4]], [github.com/antirez/ds4](https://github.com/antirez/ds4)*
*Last updated: 2026-05-18*
