# FreeToken — Edge-Native MoE Serving Engine

FreeToken is an edge-native serving engine that runs 290B+ frontier MoE models on a single consumer GPU by treating the whole machine — GPU, CPU, host RAM, and PCIe interconnect — as one elastic inference platform. Its headline move is bandwidth-adaptive CPU–GPU co-execution: it measures how fast the CPU can run MoE layers versus how fast PCIe can fetch expert weights, and splits each decode step between the two so they finish at the same time. This is the same "attack the memory term" thesis as [[MiMo-V2.5-Pro-UltraSpeed]], but aimed at a gaming PC rather than an 8-GPU node — the engine that tries to make datacenter-class MoE inference a thing you run at home.

---

## Architecture

FreeToken is a single-process, monolithic engine in the [[Local and Open Source Inference|local serving]] tradition of vLLM/SGLang rather than a distributed system. The layers, top to bottom:

- **Scheduler** (`scheduler.py`, `cache.py`) — continuous batching with overlap scheduling; consumes a message queue (UserMsg / AbortBackendMsg / CacheRebuildBackendMsg), detects tool-call anchors, and drives KV-cache management.
- **Engine** (`engine.py`) — builds the model on the `meta` device, loads weights, initializes the MoE offload cache, KV pool, page table, attention/MoE backends, sampler, and the CUDA-graph decode runner. The massive `_adjust_config` is the config-reconciliation choke point: dense models drop MoE knobs; MoE models default to offload (upgraded to hybrid only if a `benchbw` profile says it pays); specific models force page-size constraints (DeepSeek-V4 Flash → page_size 128; sliding-window/MLA/DSA → page_size 1).
- **Graph/Model** — model classes per family; decode is captured into CUDA graphs (`GraphRunner`, `cuda_graph_bs`) with two-stream overlap scheduling.
- **Daemon + servers** — Anthropic/OpenAI-compatible HTTP APIs so Codex, Claude Code, OpenCode, and other tool-calling agents connect unmodified.

The core abstraction is the **unified expert slot cache**: every expert in every layer lives in a pinned host-RAM bank; the GPU holds a fixed-size slot cache addressed by a flat id `layer_id * num_experts + expert`, with `slot_for_id`/`id_of_slot` maps and a global LRU eviction policy. Dense weights and KV pages live in their own pools, and a `rebuild_runtime_cache` path can resize all of them in place — with validation-before-destructive-free and rollback to the prior geometry on failure.

## Key techniques

**Bandwidth-adaptive CPU–GPU co-execution (the $q^\star$ policy).** Decode is bandwidth-bound ([[Theoretical LLM Inference Bottlenecks]] derives why: batch-1 arithmetic intensity ≈ 1 FLOP/byte). FreeToken's answer is not to fetch everything faster but to fetch *less*: on each step, a fraction $q^\star$ of cache misses is gathered over PCIe while the rest of that layer's compute runs on the CPU via a C++ `CpuMoeExecutor` (ISA-tiered scalar/avx2/avx512/avx512bf16), with a `cudaLaunchHostFunc` handshake to overlap the two. The split fraction is set so the PCIe fetch and the CPU compute finish together.

**`benchbw` measures reality before committing.** Rather than assuming CPU is always worth it, `ft bench bw` runs the *actual* kernels — a CPU MoE GEMV and the PCIe gather (`fast_index_copy`) — plus contended and overlapped variants, and recommends hybrid only when `cpu_bw > 2 × pcie_bw`. Results are cached to `benchbw.json`, so the auto-configuration is derived from the specific machine, not a spec sheet.

**Full-layer double-buffered prefill.** Prefill borrows `2 × num_experts` slots from the unified cache and pipelines the next layer's load against the current layer's compute. Hits are split from misses: resident experts go through a device-to-device gather (`prefill_hit_d2d`), while misses stream via `cudaMemcpyBatchAsync` (`fast_index_copy_multi_jit`), with a `_SMALL_BANK_FEAT_BYTES = 256KB` threshold to avoid sync degradation on tiny banks.

**FTW checkpoint format.** A custom O_DIRECT-friendly weight format: every tensor 4096-aligned and padded, sharded at 8 GiB, read by chunked multi-threaded O_DIRECT pread with an mmap fallback. Two `kind` tags separate dense weights (`weight`) from offload banks + alphas (`experts_bank`), so the engine can stream only what a given backend needs.

**Semantic anchor checkpoints.** For agentic workloads the differentiator: `toolcall_anchor_len` freezes the recurrent (GDN/mamba) state at the token that opens a tool call, so subsequent context edits — the tool output, thinking blocks — recompute only the delta instead of the whole prompt. Ping-pong snapshot slots and copy-on-write restore on prefix hits back this up.

**Tiered radix KV caches.** `HybridRadixCache` (snapshots recurrent state at chunk boundaries), `SWARadixCache` (sliding-window with tombstone/adopt semantics and out-of-window eviction), and a plain radix tree, all over page-aligned free slots with a `lazy_free_region` for deferred frees.

**CUDA-graph decode + elastic memory.** Decode runs under captured CUDA graphs; `rebuild_runtime_cache` re-allocates MoE slots / KV pages / mamba slots / SWA window in place and re-captures the graphs, so the VRAM split between expert cache and KV memory is tunable at runtime without an engine restart.

## Design decisions

**Correctness-first, defensive by default.** The elastic-resize path validates the new geometry before any destructive free and rolls back to the prior one on failure. The auto-config path never picks the fused/offload-first option unless measurement justifies it — conservative defaults that treat the machine's actual bandwidth as the source of truth.

**Offload is the default; hybrid is an opt-in upgrade.** MoE models default to host-RAM offload + GPU slot cache. The CPU-execution backend is activated only when `benchbw` shows CPU bandwidth beating 2× PCIe — the authors gate the exotic path behind evidence rather than shipping it always-on.

**Model-shape constraints are encoded, not papered over.** `_adjust_config` pins page_size per architecture (128 for DeepSeek-V4 Flash's paged attention, 1 for sliding-window/MLA/DSA). These constraints are real consequences of the attention shapes; forcing a single page size would silently break correctness.

**A bespoke weight format as a first-class product decision.** FTW exists because existing formats assume weights fit in VRAM or stream naively from disk; O_DIRECT alignment and sharding exist to make host-RAM→GPU streaming fast and page-cache-friendly. It's the same "design the format for the memory tier it lives on" instinct behind [[AirLLM]]'s per-layer disk split, but aimed at bandwidth rather than capacity.

## Comparison notes

**vs [[AirLLM]].** AirLLM streams one *dense layer* from disk at a time to run the biggest possible model on the smallest GPU, accepting constant disk I/O. FreeToken streams *experts* from host RAM into an LRU slot cache and only exercises CPU compute when measured bandwidth says it wins. AirLLM optimizes capacity at any latency cost; FreeToken optimizes interactive speed on a machine that already has the RAM.

**vs llama.cpp / GGUF.** llama.cpp quantizes the whole model and runs CPU or CPU+GPU split; FreeToken keeps frontier quant formats (nvfp4, mxfp4, fp8, q4_0) but treats offload as the primary memory strategy rather than quantization alone. They attack the same [[Theoretical LLM Inference Bottlenecks|bandwidth ceiling]] from the quantization side versus the memory-hierarchy side.

**vs vLLM / SGLang.** FreeToken borrows vLLM/SGLang's continuous batching, PagedAttention, and radix caching (it cites mini-sglang as the direct inspiration) but reorients the whole engine around a heterogeneous single box instead of a fleet of H100s. vLLM assumes the weights fit in VRAM; FreeToken's premise is that they don't, and that the CPU and PCIe bus are co-equal compute resources.

**vs [[MiMo-V2.5-Pro-UltraSpeed]].** Both are MoE model-system codesign: MiMo/TileRT used selective FP4 quantization + DFlash speculation + persistent kernels to hit 1000+ tok/s on an 8-GPU node. FreeToken pursues the same bandwidth-attack goal with a different lever — CPU co-execution and expert offload — on a single consumer GPU. MiMo codesigns the *model*; FreeToken codesigns the *hardware's* memory hierarchy.

## Tags

#tool #project #inference #moe #local-inference

---
*Sources: [[raw/freetoken]], [[summary/freetoken]]*
*Last updated: 2026-08-22*
