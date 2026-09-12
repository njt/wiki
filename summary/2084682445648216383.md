---
url: https://xcancel.com/Pokee_AI/status/2084682445648216383
title: "Pokee-Isaac 28B — World's First 10M-Token Context Agentic Model on a Single GPU"
author: Pokee AI (@Pokee_AI)
date_published: 2026-08-04
topics:
  - ai-research-and-models
---

Pokee AI announced Pokee-Isaac 28B, a 28-billion-parameter model they describe as the world's first real 10M-token context frontier-class agentic model deployable on a single GPU (starting from RTX 4090). The model uses what Pokee calls a "proprietary non-decoder-only architecture" — partially fine-tuned from Qwen3.6-27B under Apache 2.0, but with other weights trained from scratch. It is closed-source and API-only.

On benchmarks, Pokee claims 93.3% on RULER at 10M tokens, up to 137K tokens/s prefill on a single B200 at full context, and leadership on BFCL v4, τ³-bench, and the DTAP security red-teaming benchmark (lowest combined attack success rate among evaluated models). Pricing is $0.15/M input tokens and $1/M output tokens, with deployment available in VPC, on-premises, or on-device via Day-0 vLLM and SGLang support.

The announcement thread is notable for its format: nearly every reply from Pokee AI offers 2,000 free API credits, making the thread read more like a growth campaign than a technical launch. One skeptical reply from @specimba questioned whether the product is "benchmark optics + API-credit farming" rather than a broadly useful deployment path, noting the ~173s TTFT and B200-class hardware requirements undercut the single-GPU narrative. Pokee AI's response was to offer more free credits. No independent benchmarks or third-party verification were provided.
