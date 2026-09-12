---
url: https://github.com/lyogavin/airllm
title: AirLLM
author: Gavin Li (lyogavin)
date_fetched: 2026-07-03
date_published: 2023-11-20
topics:
  - misc
---

# AirLLM — Full Repository Analysis

## Overview

AirLLM is a Python library that dramatically reduces GPU memory usage for LLM inference by keeping only one transformer layer on the GPU at a time. It can run 70B models on a single 4GB GPU, 405B Llama 3.1 on 8GB, and 671B DeepSeek-V3 on ~12GB — without quantization, distillation, or pruning. Optional block-wise weight compression can add 3x speedup with negligible accuracy loss.

The project was created by Gavin Li, building on work by SimJeg from a Kaggle competition. It has 10k+ GitHub stars and is available as `pip install airllm`.

## Repository Structure

```
airllm/
├── README.md                    # Main documentation
├── README_ja.md                 # Japanese translation
├── requirements.txt             # Training dependencies
├── setup.py                     # Package: v3.0.1, torch>=2.4, transformers>=4.49
├── funding.json                 # GitHub Sponsors config
├── air_llm/
│   ├── setup.py                 # Package setup
│   ├── __init__.py              # Conditional imports (Mac vs Linux)
│   ├── inference_example.py     # Usage example
│   ├── airllm/
│   │   ├── __init__.py          # Defensive imports of model-specific classes
│   │   ├── airllm_base.py       # CORE: streaming inference engine (387 lines)
│   │   ├── airllm.py            # AirLLMLlama2 (trivial subclass)
│   │   ├── auto_model.py        # AutoModel: detects architecture, picks class
│   │   ├── utils.py             # Layer splitting, compression, disk I/O (445 lines)
│   │   ├── profiler.py          # Timing/memory tracking (31 lines)
│   │   ├── airllm_mistral.py    # Mistral support (trivial subclass)
│   │   ├── airllm_mixtral.py    # Mixtral MoE support (trivial subclass)
│   │   ├── airllm_qwen.py       # QWen 1 support
│   │   ├── airllm_qwen2.py      # QWen 2 support (trivial subclass)
│   │   ├── airllm_chatglm.py    # ChatGLM custom architecture
│   │   ├── airllm_baichuan.py   # Baichuan custom architecture
│   │   ├── airllm_internlm.py   # InternLM custom architecture
│   │   ├── airllm_llama_mlx.py  # MacOS/MLX path: full reimplementation (436 lines)
│   │   ├── tokenization_baichuan.py
│   │   └── persist/
│   │       ├── __init__.py
│   │       ├── model_persister.py       # Abstract factory: platform→persister
│   │       ├── safetensor_model_persister.py  # Linux: safetensors per-layer I/O
│   │       └── mlx_model_persister.py   # Mac: NPZ per-layer I/O with name mapping
│   ├── tests/
│   │   ├── test_compression.py          # Compression roundtrip correctness
│   │   ├── test_automodel.py            # Architecture→class dispatch table
│   │   ├── test_streaming_gpu.py        # Full e2e test harness with VRAM capping
│   │   └── test_notebooks/              # Interactive test notebooks
│   └── examples/
│       ├── run_all_types_of_models.ipynb
│       ├── run_llama3.1_405B.ipynb
│       └── run_on_macos.ipynb
├── training/                   # Fine-tuning utilities (QLoRA)
├── rlhf/                       # DPO training
├── anima_100k/                 # Training data generation
├── eval/                       # ELO tournament evaluation
├── data/                       # Evaluation datasets
├── scripts/                    # Utility scripts
└── assets/                     # README images
```

### Line counts of core files

| File | Lines |
|------|-------|
| `utils.py` | 445 |
| `airllm_llama_mlx.py` | 436 |
| `airllm_base.py` | 387 |
| `mlx_model_persister.py` | 112 |
| `auto_model.py` | 60 |
| `safetensor_model_persister.py` | 37 |
| `model_persister.py` | 38 |
| `profiler.py` | 31 |
| **Total core** | **~1,546** |

The entire inference engine is under 1,600 lines of Python. The slimness is a feature.

## Architecture

### The Core Idea: Layer Streaming

AirLLM's architecture is built on a single insight: **transformers process layers sequentially, so you only need one layer's weights on the GPU at any given moment**.

The full model is constructed on the `meta` device (via `accelerate.init_empty_weights`) — no memory allocated. Then, forward hooks are installed on every major module (embedding, each decoder layer, final norm, lm_head). Right before each module executes, its weights are streamed from disk to GPU via a `register_forward_pre_hook`. Right after, they are moved back to `meta` via a `register_forward_hook`, freeing VRAM.

This means AirLLM delegates the entire forward pass to the real `transformers` model — it doesn't reimplement attention, rotary embeddings, or KV-cache management. The streaming layer is just a transparent memory-management wrapper. New model architectures work as soon as HuggingFace `transformers` supports them.

### Component Breakdown

**1. `AirLLMBaseModel` (`airllm_base.py`)** — The core engine:
- Constructs the real `AutoModelForCausalLM` on `meta` device (line 183-194)
- Installs forward pre/post hooks on each layer (line 342-376)
- Pre-hook: loads layer from disk via `load_layer()`, moves to GPU; optionally triggers async prefetch of next layer on `ThreadPoolExecutor` (line 363-367)
- Post-hook: moves layer back to `meta`, calls `clean_memory()` (line 369-376)
- Handles tied embeddings (line 332-336): keeps embeddings resident, re-ties lm_head
- Handles FP8 pre-quantized weights: passes `float8_e4m3fn` tensors through verbatim without dtype casting (line 292-293), a critical detail — casting an FP8 weight to FP16 silently drops the quantization
- Handles bitsandbytes on-the-fly quantization via `AutoHfQuantizer`
- Patches model's `.device` property with a runtime subclass so `transformers`' generation code places tensors on CUDA even though parameters live on `meta` between layers (line 218-232)

**2. `split_and_save_layers()` (`utils.py`)** — Preprocessing step:
- Reads the HF checkpoint's weight map (index.json or single-file detection)
- Iterates shard-by-shard, extracting each layer's weights
- Optionally applies 4-bit or 8-bit block-wise quantization on-disk (reduces I/O, not compute)
- Saves each layer as a separate safetensors file with a `.done` marker for atomicity
- Supports `delete_original` to reclaim disk space after splitting
- Handles fp8 companion tensors (`weight_scale_inv`) by loading all shards a layer spans

**3. `AutoModel` (`auto_model.py`)** — Architecture dispatch:
- Reads `config.architectures` from HuggingFace config
- Maps known non-standard architectures (ChatGLM, QWen, Baichuan, InternLM) to dedicated subclasses via `ARCH_OVERRIDES` dict
- Everything else uses the generic `AirLLMBaseModel` — newly released models work zero-code
- On macOS, always routes to `AirLLMLlamaMlx`

**4. Persistence Layer (`persist/`)** — Platform abstraction:
- `ModelPersister` is a singleton factory: detects macOS → `MlxModelPersister`, else → `SafetensorModelPersister`
- `SafetensorModelPersister`: per-layer safetensors files with `.done` marker for resume
- `MlxModelPersister`: per-layer `.mlx.npz` files + name mapping (Llama→MLX conventions: `mlp→feed_forward`, `q_proj→wq`, etc.)

**5. macOS/MLX Path (`airllm_llama_mlx.py`)** — Full reimplementation:
- On Apple Silicon, AirLLM switches to a completely different engine using MLX
- Re-implements TransformerBlock, Attention, RMSNorm, FeedForward in MLX
- Implements a manual generate loop with KV-cache management (line 265-437)
- Loads per-layer weights via `MlxModelPersister`, applies name mapping, does `tree_unflatten`
- Explicitly deletes each layer after use with `del` + `gc.collect()` — MLX lazy evaluation means manual memory management is essential

### The Prefetching Trick

When prefetching is enabled (`airllm_base.py` line 363-367), the pre-hook for layer N submits the load of layer N+1 to a `ThreadPoolExecutor(max_workers=1)`. The post-hook for layer N doesn't block — it just moves N to meta. When the pre-hook for layer N+1 fires, it checks if the prefetch future is ready and uses the result directly instead of loading again. This overlaps disk I/O with GPU compute, giving ~10% speed improvement.

### Compression vs Quantization

AirLLM's "compression" (4-bit or 8-bit) is different from standard quantization. Standard quantization quantizes both weights AND activations at runtime to accelerate compute. AirLLM only quantizes the weights on disk — the bottleneck is loading them into VRAM, not the compute itself. At runtime, weights are dequantized back to FP16 before the layer runs. This is simpler to implement and safer for accuracy because you don't need to handle activation outliers.

The compression roundtrip is tested in `test_compression.py`: random tensors are compressed, decompressed, and checked for RMSE < 0.1.

## Key Techniques

### 1. Meta-device + Forward Hooks = Zero-VRAM Model Construction

The `init_empty_weights(include_buffers=False)` call from `accelerate` creates every parameter as a `torch.empty(0)` on the `meta` device. The model object exists but consumes no GPU memory. Forward hooks then materialize parameters on demand.

This is significantly different from other offloading approaches (like DeepSpeed ZeRO-Infinity) which require the full model in CPU RAM and swap pages to GPU. AirLLM's approach only requires disk space, not CPU RAM.

### 2. Layer-Shard Split as Preprocessing

Instead of loading the full checkpoint and then slicing it in memory, `split_and_save_layers()` streams through shard files incrementally, extracts each layer's weights, and writes them out as individual files. This means you can split a 405B model on a machine that couldn't load it into RAM — a critical practical detail.

The `.done` marker pattern enables resumability: if the splitting process crashes, re-running it skips already-completed layers.

### 3. Tied Embeddings Handling

For models with `tie_word_embeddings=True` (no separate lm_head weight), AirLLM keeps the embedding layer permanently on GPU (it's small) and re-ties lm_head to it. The `_streamed_indices` excludes both the embedding and lm_head from the streaming cycle for these models (line 332-338).

### 4. FP8 Verbatum Pass-Through

Pre-quantized FP8 weights (like in DeepSeek-V3) carry companion `weight_scale_inv` tensors. AirLLM detects `torch.float8_e4m3fn` dtypes and skips the normal dtype-casting path (line 292-293), placing them on device verbatim. Without this, the FP8 quantization is silently destroyed and output is garbage.

### 5. Device Property Patching

The model's `.device` and `.dtype` properties are overridden at runtime (line 218-232) via a dynamic subclass. This is necessary because `transformers`' generation code checks `model.device` to decide where to place input tensors and KV-cache tensors. Since parameters are on `meta` between layer executions, the model would otherwise report `device=meta` and break generation.

### 6. Platform-Specific Persistence

The persistence layer is an abstract factory: `ModelPersister.get_model_persister()` returns a platform-specific implementation. On Linux, it's safetensors (`.safetensors` + `.done`). On macOS, it's NPZ (`.mlx.npz` + `.mlx.done`) with a comprehensive name-mapping function (`map_torch_to_mlx`) that translates Llama weight names to MLX conventions throughout.

### 7. Thread-Per-Layer Prefetching

A single `ThreadPoolExecutor` worker prefetches the next layer's weights from disk while the current layer is computing. This is simple but effective — the `.pin_memory()` call on line 274-275 ensures the loaded tensors are in pinned (page-locked) memory for fast async CUDA transfers.

## Design Decisions

### Optimized For: Minimum VRAM

Everything flows from this priority. AirLLM optimizes for running the largest possible model on the smallest possible GPU. It sacrifices:

- **Speed**: Disk I/O dominates. Even with prefetching, you're waiting for safetensors to load from disk for every layer. The README reports ~10% improvement from prefetching, not 2x.
- **Latency on first run**: The splitting step converts a multi-GB checkpoint into per-layer files. This is a one-time cost per model.
- **Disk usage**: Up to 2x the original model size (original + split), unless `delete_original=True`.

### Sacrificed: Throughput

This is NOT a high-throughput inference server. It's designed for running one query at a time on a single GPU with minimal VRAM. For batch inference or serving, you'd want vLLM or TensorRT-LLM, which use PagedAttention and continuous batching — techniques that optimize for throughput, not memory.

### Sacrificed: Training

AirLLM only handles inference. The layer-streaming pattern doesn't work for training because backpropagation requires all activations. The `training/` and `rlhf/` directories contain separate (standard) training scripts — they don't use AirLLM's streaming.

### Smart: Delegating to Transformers

The v3.0 redesign delegates the entire forward pass to HuggingFace `transformers`. Earlier versions reimplemented attention, rotary embeddings, and generation logic per model family. By switching to hooks on a real `AutoModelForCausalLM`, AirLLM gained support for all new model architectures with zero code changes. The only reason for dedicated subclasses now is non-standard module layouts (e.g., ChatGLM doesn't use `model.layers` naming).

### Smart: Separate macOS Engine

Rather than forcing MLX into the PyTorch-based streaming architecture, AirLLM has a completely separate engine for macOS (`airllm_llama_mlx.py`). This is a full reimplementation using MLX's native tensor operations. It's the right call — the Python/CUDA memory management tricks that work on Linux don't translate to Apple Silicon, and MLX's lazy evaluation model requires different resource management (explicit `mx.eval()` calls to force computation, explicit `del` + `gc.collect()` to free memory).

### Clever: Device Property Deception

Patching `model.device` to lie about where parameters live is a hack, but it's an elegant hack. The alternative would be forking `transformers`' generation loop. By making the model report `device=cuda` even when parameters are on `meta`, AirLLM inherits all of `transformers`' generation features — beam search, sampling, stopping criteria, cache management — for free.

## Comparison Notes

### vs llama.cpp / GGUF

`llama.cpp` quantizes models to 4-bit integers and runs them entirely in CPU RAM or offloaded across CPU+GPU. AirLLM takes the opposite approach: keep weights at full (or near-full) precision on disk and stream them to GPU one layer at a time. The trade-off is accuracy vs latency — AirLLM preserves full model quality but is slower per token because of constant disk I/O. llama.cpp quantizes aggressively (with quality loss) but is fast enough for interactive use.

### vs DeepSpeed ZeRO-Inference

DeepSpeed ZeRO-Inference offloads model weights from GPU to CPU RAM, pinning them in memory and streaming them to GPU as needed. AirLLM offloads to **disk**, not CPU RAM, which is the critical difference. ZeRO-Inference requires enough CPU RAM to hold the entire model (e.g., 140GB for a 70B model in FP16). AirLLM needs only enough disk space — which is abundant and cheap.

### vs vLLM / TensorRT-LLM

These are high-throughput inference servers. They keep the whole model in GPU VRAM and optimize for serving many concurrent requests through PagedAttention, continuous batching, and kernel fusion. AirLLM is for the opposite use case: one user, one GPU, one query at a time, with the model too big to fit in VRAM.

### vs Petals / Distributed Inference

Petals splits models across multiple machines in a peer-to-peer network. AirLLM is strictly single-machine. No network dependency, no coordination overhead, but also no scaling beyond what one machine's disk can hold.

### In the Wiki Context

AirLLM fits the [[Local and Open Source Inference]] ecosystem as the "run anything, slowly" option. It's the practical answer to "can I try a 405B model on my gaming GPU?" — where llama.cpp says "no, that won't fit even quantized" and vLLM says "you need 8×A100s." [[LocalAI]] provides the API layer; AirLLM provides the model-fitting layer. Together they enable a fully local stack on modest hardware.

The key insight connecting AirLLM to the broader wiki themes: **the bottleneck for local inference is not compute, it's memory bandwidth and capacity** (as [[Inference Cost Napkin Math]] documents). AirLLM's approach of disk-streaming is extreme — trading bandwidth for capacity — but it proves that the memory-capacity constraint can be routed around entirely.

## Critical Analysis

### Strengths

- **Genuinely novel approach**: Layer-streaming from disk, controlled via forward hooks on a meta-device model, is not something any other popular inference tool does
- **Surprisingly simple**: The core engine is ~400 lines of Python. The complexity lives in the preprocessing (splitting, compression) and platform adaptation, not the streaming logic itself
- **Model-agnostic by design**: By delegating to `transformers`, new architectures work without code changes. This is a real advantage over llama.cpp, which needs per-architecture reimplementation
- **Practical**: The disk-space check, resumable splitting, `.done` markers, and `delete_original` option show attention to real-world deployment friction

### Weaknesses

- **Speed**: Disk I/O per layer per token makes this unusable for interactive chat. It's for batch processing, experimentation, and testing — not production serving
- **Single-query**: No concurrency model. You can't batch requests or serve multiple users
- **Python-only**: No C++/Rust inference engine. The dependency on PyTorch + `transformers` + `accelerate` is heavy compared to llama.cpp's standalone binary
- **Documentation debt**: The README is marketing-heavy; the code has good comments added in v3.0 but earlier parts are sparsely commented. The training/rlhf/eval directories are essentially unmaintained artifacts
- **No streaming output**: `generate()` returns all at once. There's no token-by-token streaming, which limits integration with chat UIs
- **Mac path fragility**: The MLX reimplementation hardcodes Llama architecture and doesn't benefit from the generic `transformers` delegation pattern

### The Who-Is-This-For Question

AirLLM is for ML practitioners who want to experiment with large models on consumer hardware. It's not for production deployment. It's for the researcher with a single 3090 who wants to see what DeepSeek-V3 outputs look like before deciding whether to rent cloud GPUs for a real run.
