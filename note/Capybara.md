# Capybara

A unified visual creation model from ByteDance that handles text-to-image, text-to-video, and instruction-based editing of both. One model for generation and manipulation across modalities.

---

## Key Themes

#ai #image-generation #video-generation #diffusion-models

The "unified" claim is the interesting part. Most visual AI tools do one thing -- generate images from text, or edit existing images, or generate video. Capybara combines text-to-image, text-to-video, instruction-based image-to-image, and instruction-based video-to-video in a single model. That means you could generate a video from a text prompt, then edit specific frames with natural language instructions.

The FP8 quantization for memory optimization and distributed multi-GPU inference suggest this is designed for serious production use, not just research demos. ComfyUI integration with custom nodes means it plugs into existing creative workflows.

## Critical Analysis

MIT licensed, which is unusually permissive for a model of this scope from ByteDance. The architecture (diffusion models + transformers, SIGLIP vision encoder, multiple text encoders) is complex -- this isn't something you'll run casually on a laptop.

The real test is quality. Unified models historically sacrifice per-task quality for versatility. Whether Capybara's image generation matches dedicated image models, or its video matches dedicated video models, is the question the README doesn't answer. Benchmark numbers would help.

Connects to [[Dolphin]] as another ByteDance contribution to the open-source AI ecosystem. ByteDance is quietly releasing production-grade models at a pace that rivals Meta's open-source strategy.

---
*Sources: [[summary/capybara]]*
*Last updated: 2026-05-14*
