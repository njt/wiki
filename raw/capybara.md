---
title: "Capybara"
url: https://github.com/xgen-universe/Capybara
date_fetched: 2026-05-14
section: "Random"
---

# Capybara: Unified Visual Creation Model

Comprehensive visual generation and editing framework built on diffusion models and transformer architectures. Enables high-quality visual synthesis and manipulation tasks across multiple modalities.

## Supported Tasks
- Text-to-Video (T2V)
- Text-to-Image (T2I)
- Instruction-based Video-to-Video (TV2V)
- Instruction-based Image-to-Image (TI2I)

## Key Technical Features
- Multi-task support across generation and instruction-based editing
- Distributed inference for efficient multi-GPU processing
- FP8 quantization for memory optimization
- ComfyUI integration with custom nodes
- Automatic prompt enhancement via Qwen3-VL-8B-Instruct

## Architecture
- Scheduler configuration
- Text encoders (byt5-small, Glyph-SDXL-v2, LLM components)
- Transformer module (Capybara v0.1)
- VAE for image encoding
- Vision encoder (SIGLIP)

PyTorch 2.6.0 with CUDA 12.6 support. Python 3.11. MIT License.
