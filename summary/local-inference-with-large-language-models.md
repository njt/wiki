---
url: https://jacquesmattheij.com/local-inference-with-large-language-models/
title: "Local Inference with Large Language Models"
author: Jacques Mattheij
date_fetched: 2026-11-03
date_published: not stated (2026)
topics:
  - local-and-open-source-inference
  - ai-infrastructure-and-hardware
---

Jacques Mattheij's long field guide to running LLMs on your own hardware, motivated by his M&A advisory business's NDA obligations: confidential client reports cannot leave the building, so proofreading — the workload AI is best at — has to happen in-house. He concludes a local rig is worth the money even though hosted providers are always cheaper.

The guide covers the three pillars: inference engines, models, and hardware. Engines reviewed from production use are llama.cpp (reliable, simple, auditable), vLLM (fastest but complex and fragile), SGLang (middle ground, Docker complicates auditing), and two specialists — DwarfStar (antirez's Mac/DeepSeek-focused engine) and Ninfer (squeezes an RTX 5000-series GPU). His current workhorses are all Chinese open-weight models: Qwen 3.8 27B for coding, DeepSeek, and above all GLM 5.3, a "extremely capable" large model used for proofreading that "has never refused an assignment." He is blunt that the reason is economics and openness: China releases commercial models free, and the US response has been fearmongering and regulatory capture attempts.

The hardware section is the meat: a tiered cost breakdown (hobby ~$10k, small business more, enterprise $5–10k per seat), detailed opinions on the RTX 3090 (still the value king, used prices above original retail), the single-slot RTX PRO 4000 Blackwell, RTX 4090s with Chinese 48GB remanufacturing, and two DGX Sparks (impressive specs, but 275 GB/s bandwidth is a third of a 3090). Beyond one box he covers PCIe lane splitting, risers, and PCIe switch boards (ADT) as the way to mount more GPUs than a motherboard can hold. The finished rig: ~$25k, 8 GPUs, 240GB VRAM, 3kW — thousands of tokens/sec on concurrent Qwen3.8s, 50–100 tok/s on GLM 5.3, "like having a very fancy space heater."
