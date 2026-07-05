---
url: https://imil.net/blog/posts/2026/rtx-5080-+-rtx-3090-setup-80+-tok-s-on-qwen-3.6-27b-q8/
title: RTX 5080 + RTX 3090 Setup — 80+ tok/s on Qwen 3.6 27B Q8
author: iMil
date_fetched: 2026-07-05
date_published: 2026-06-13
site: imil.net
tags: [ai, llm, gpu, local-inference, llama.cpp]
---

# Background

The author (iMil) initially bought an RTX 5080 for gaming and AI. By 2026, needing more than 16GB VRAM for models like Qwen 3.5, Gemma, and Qwen 3.6, they added a refurbished RTX 3090 (24GB). This allowed Qwen 3.6 Q4 quants at ~30 tok/s, later 50-60 with MTP. Unsatisfied with the 5080 sitting mostly idle, they pursued a dual-GPU setup.

**Motherboard:** Asus Prime X570-Pro — the "Pro" designation is crucial because it allows splitting the 16x PCIe into 2x8 lanes.

**Riser:** A quality PCIe 4 riser for the 5080 in the second slot.

---

# BIOS Configuration

Crucial finding: "you CAN'T boot the OS in BIOS/MBR mode" — this prevents dual-card usage entirely.

Settings applied:

| Tab | Path | Setting |
|-----|------|---------|
| Boot | CSM | Disabled |
| Advanced -> PCI Subsystem | Above 4G Decoding | Enabled |
| Advanced -> PCI Subsystem | ReSize BAR Support | Auto or Enabled |
| Advanced | PCIEX16_1 Link Mode | Gen 4 |
| Advanced | PCIEX16_2 Link Mode | Gen 4 |

---

# Kernel & Driver Setup

The author points to NVIDIA's Tesla driver installation docs, noting the documentation is messy.

They tested open-gpu-kernel-modules but it failed because the two GPUs are different models and generations. For those with matching cards, the patched driver requires:
1. Uninstalling nvidia-dkms-open
2. Blacklisting the nova driver

The expected p2p status check via `nvidia-smi topo -p2p r` should show "OK" between both GPUs.

For mismatched cards like this setup, the standard `nvidia-open` driver is used. The nvidia-smi output confirms:
- GPU 0: RTX 3090, bus 07:00.0, 24,576 MiB total, 23,646 MiB used
- GPU 1: RTX 5080, bus 08:00.0, 16,303 MiB total, 15,861 MiB used
- Driver: NVIDIA-SMI 610.43.02, KMD Version 610.43.02, CUDA UMD Version 13.3
- Both show P8 power state, low temperatures (34C / 31C)

---

# llama.cpp Build & Configuration

## CMake Build Flags

```
cmake -B build -DBUILD_SHARED_LIBS=OFF -DGGML_CUDA=ON -DGGML_NATIVE=ON
-DGGML_CUDA_FA=ON -DGGML_CUDA_FA_ALL_QUANTS=ON
-DCMAKE_CUDA_ARCHITECTURES="86;120"
-DCMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc -DGGML_CUDA_NCCL=OFF
```

Key details:
- `CMAKE_CUDA_ARCHITECTURES="86;120"` enables both Ampere (86 = RTX 3090) and Blackwell (120 = RTX 5080)
- `-DGGML_CUDA_NCCL=OFF` — the author found NCCL counterproductive despite what llama-server logs claim
- Flash attention is enabled for all quants

## Server Launch Command

```
llama-server -m ./models/Huihui-Qwen3.6-27B-abliterated-ggml-model-Q8_0.gguf \
    -c 229376 \
    -np 1 -fa on -ngl 99 -ub 512 -t 6 --no-mmap \
    --temp 0.7 --top-p 0.8 --top-k 20 --min-p 0.0 \
    --presence-penalty 0.0 --repeat-penalty 1.0 \
    -ctk q8_0 -ctv q8_0 --kv-unified \
    --chat-template-kwargs {"preserve_thinking": true} \
    --spec-type ngram-mod,draft-mtp --spec-draft-n-max 3 \
    -sm tensor -ts 2,3 \
    --port 8001 --host 0.0.0.0
```

### Key Configuration Details

- **Model:** Huihui-Qwen3.6-27B-abliterated-ggml-model-Q8_0.gguf from huihui-ai on HuggingFace
- **Context length:** 229,376 tokens
- **KV cache quant:** q8_0 for both keys and values, with --kv-unified
- **Offloaded layers:** -ngl 99 (all layers to GPU)
- **Batch size:** 512 tokens (-ub 512)
- **CPU threads:** 6
- **No mmap** is used
- **Speculative decoding:** dual approach using ngram-mod and draft-mtp with max 3 draft tokens
- **Tensor split strategy:** -sm tensor per llama.cpp multi-GPU docs, with -ts 2,3 ratio to fill both card's VRAM
- The Q8 quantization of this 27B model fits in the combined ~39GB VRAM along with a 230k context at q8 KV cache

---

# Performance Results

The setup delivers "80+ tokens/sec" with peaks above 90 depending on task.

| Metric | Value |
|--------|-------|
| Generation speed (100 tokens) | 81.84 t/s |
| Generation speed (388 tokens) | 91.13 t/s |
| Prompt eval | 219.76 ms / 17 tokens (77.36 t/s) |
| Total eval | 5,185.10 ms / 457 tokens (88.14 t/s) |
| Total time | 5,404.85 ms / 474 tokens |
| Draft acceptance rate | 0.77295 (320 accepted / 414 generated) |

### Speculative Decoding Statistics

- **ngram-mod:** 1,169 gen drafts, 1,169 acc drafts, 74,496 gen tokens, 44,050 acc tokens
- **draft-mtp:** 42,477 gen drafts, 36,208 acc drafts, 127,431 gen tokens, 86,553 acc tokens
- Graphs reused: 41,669

---

# PCIe Verification

To confirm cards run at full speed during workload:

```bash
sudo lspci -vvv -s 07:00.0 | grep "LnkSta:"
```

Expected output for each port with the 16x/2 split:

"LnkSta: Speed 16GT/s, Width x8 (downgraded)"

This confirms both cards are operating at PCIe Gen 4 speeds with 8 lanes each (the "downgraded" note simply reflects that the slot supports 16x electrically but is wired for 8x in this dual configuration).
