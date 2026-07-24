# AirLLM

AirLLM is a Python library (~1,500 lines) that runs 70B+ LLMs on consumer GPUs by streaming one transformer layer from disk at a time. It can run 405B Llama 3.1 on 8GB VRAM and 671B DeepSeek-V3 on ~12GB — without quantization, distillation, or pruning. Created by Gavin Li, building on SimJeg's Kaggle competition work. 10k+ GitHub stars, Apache 2.0, `pip install airllm`.

The core insight: transformers process layers sequentially, so you only ever need one layer's weights on the GPU. Everything else lives on disk or the `meta` device. This is a different axis of the memory-capacity trade-off than the rest of the local inference ecosystem explores — it trades speed (constant disk I/O) for model scale (any model fits if you have the disk space).

Compare with [[Bonsai 27B]], which takes the opposite approach: extreme quantization (1.125 bpw binary weights) that keeps the entire 27B model in 3.9 GB of memory at full speed, rather than streaming layers. Both solve the "big model, small hardware" problem from different architectural directions — temporal vs. spatial decomposition.

---

## Architecture

AirLLM's engine is built on **forward hooks on a meta-device model**. The real `AutoModelForCausalLM` from HuggingFace `transformers` is constructed on the `meta` device (zero memory, via `accelerate.init_empty_weights`). Forward hooks on each module stream weights from disk → GPU before execution and back to `meta` after:

```
embed → [disk load] → GPU → compute → [meta] →
layer_0 → [disk load] → GPU → compute → [meta] →
layer_1 → [disk load] → GPU → compute → [meta] →
...
lm_head → [disk load] → GPU → compute → [meta]
```

Because `transformers` owns the forward pass, **new model architectures work with zero code changes** — AirLLM doesn't re-implement attention, rotary embeddings, or KV-cache logic. It's a transparent memory-management wrapper.

**Key components** (`air_llm/airllm/`):

| File | Lines | Role |
|------|-------|------|
| `airllm_base.py` | 387 | Core engine: meta-model construction, hook installation, layer streaming, prefetching |
| `utils.py` | 445 | Preprocessing: layer splitting, compression/decompression, disk space checks |
| `auto_model.py` | 60 | Architecture detection → class dispatch via `ARCH_OVERRIDES` dict |
| `airllm_llama_mlx.py` | 436 | macOS path: full MLX reimplementation with manual KV-cache loop |
| `persist/` | ~187 | Platform abstraction: safetensors (Linux) vs NPZ (macOS) per-layer I/O |

**Prefetching**: A single `ThreadPoolExecutor` worker loads layer N+1 from disk while layer N computes on GPU. Weights are placed in pinned memory for fast async CUDA transfers. ~10% speed improvement.

**Compression**: Optional 4-bit or 8-bit block-wise quantization of weights **on disk only**. Weights are dequantized to FP16 before each layer runs. This is not runtime quantization — it reduces the I/O bottleneck (smaller files to load) without touching activation quantization. Enables ~3x speedup with <0.1 RMSE loss per the test suite.

## Key Techniques

**Meta-device + hooks = zero-VRAM model construction.** `init_empty_weights(include_buffers=False)` creates every parameter as a zero-size tensor on the `meta` device. The model object exists but consumes no GPU memory. Forward hooks materialize parameters on demand via `set_module_tensor_to_device`. This means AirLLM needs only disk space, not CPU RAM — unlike DeepSpeed ZeRO-Inference which requires the full model in system memory.

**Layer-shard split as streaming-friendly preprocessing.** `split_and_save_layers()` streams through HF checkpoint shards incrementally, extracts each layer's weights, and writes individual safetensors files with `.done` markers for atomicity. This means you can split a 405B model on a machine that couldn't load it into RAM. Supports `delete_original` to reclaim disk space after splitting.

**FP8 verbatim pass-through.** Pre-quantized FP8 weights (DeepSeek-V3, Qwen3 FP8 variants) carry companion `weight_scale_inv` tensors. AirLLM detects `torch.float8_e4m3fn` dtypes and places them on device without dtype casting (`airllm_base.py` line 292-293). Casting an FP8 weight to FP16 silently destroys the quantization and produces garbage.

**Device property deception.** The model's `.device` property is patched at runtime via a dynamic subclass (`airllm_base.py` lines 218-232) to report `cuda` even though parameters live on `meta` between layer executions. This lets `transformers`' generation code place input tensors and KV-cache on the right device without forking the library.

**Platform-specific engines.** On macOS, AirLLM switches to a completely separate MLX engine (`airllm_llama_mlx.py`) — a full reimplementation of TransformerBlock, Attention, RMSNorm, and FeedForward in MLX with explicit `mx.eval()` and `del` + `gc.collect()` for manual memory management. The Linux PyTorch path and macOS MLX path share only the persistence layer.

## Design Decisions

**Optimized for minimum VRAM, not speed.** Every design choice flows from running the largest possible model on the smallest GPU. Disk I/O dominates latency — each generated token requires loading every layer from disk. Prefetching recovers ~10%, not 2x. This is not a production inference server.

**Disk offloading, not CPU RAM offloading.** DeepSpeed ZeRO-Inference and FlexGen offload to CPU RAM. AirLLM offloads to **disk** — the cheapest, most abundant storage tier. The trade-off: much slower per-token latency, but unlimited model capacity (any model fits if it fits on your SSD).

**Delegation over reimplementation.** v3.0's key architectural change: earlier versions re-implemented attention and generation per model family. v3.0 delegates to `transformers` via hooks. This means new architectures (Llama 4, Phi-4, Gemma 3) work on release day, and AirLLM doesn't need to track the accelerating pace of model releases.

**Separate macOS engine.** Rather than forcing MLX into the PyTorch streaming pattern, the macOS path is a completely separate codebase using MLX's native lazy evaluation. MLX's memory model requires explicit `mx.eval()` to force computation and explicit deletions to free memory — the PyTorch CUDA memory tricks don't apply.

**One-time preprocessing, per-use streaming.** The model must be split into per-layer files before first use. This is a one-time cost (minutes to hours depending on model size) that doubles disk usage unless `delete_original=True`. After splitting, inference starts in seconds.

## Comparison Notes

**vs llama.cpp / GGUF**: llama.cpp quantizes aggressively (4-bit) and runs on CPU or CPU+GPU split. AirLLM preserves full precision and streams to GPU. llama.cpp is faster per token and supports interactive chat; AirLLM runs larger models at higher quality. They optimize different constraints.

**vs DeepSpeed ZeRO-Inference**: Both stream weights to GPU. ZeRO-Inference requires the full model in CPU RAM (~140GB for 70B FP16). AirLLM requires only disk space. ZeRO-Inference is faster (CPU→GPU is faster than disk→GPU) but needs expensive RAM.

**vs vLLM / TensorRT-LLM**: These are high-throughput servers (PagedAttention, continuous batching, kernel fusion). They keep models in GPU VRAM and optimize for concurrency. AirLLM is single-query, single-GPU, minimal VRAM — the opposite use case.

**vs Petals / distributed inference**: [[Petals — Decentralized LLM Inference]] splits across machines peer-to-peer. AirLLM is single-machine. No network coordination, but also no horizontal scaling.

**In the wiki**: AirLLM is the extreme end of the local inference spectrum that [[Local and Open Source Inference]] surveys. Where [[LocalAI]] provides the API layer and [[Self-Hosted LLMs]] maps hardware to models, AirLLM answers "what's the absolute biggest model I can run on this card?" Like [[Inference Cost Napkin Math]] documents, memory bandwidth and capacity are the real bottlenecks — AirLLM routes around capacity constraints entirely at the cost of bandwidth.

## Tags

#tool #project #local-inference #llm #gpu #memory-optimization #python

## Source

[github.com/lyogavin/airllm](https://github.com/lyogavin/airllm) — v3.0.1, Apache 2.0. Fetched 2026-07-03.
