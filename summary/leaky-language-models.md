---
url: https://arxiv.org/html/2607.20723v1
title: "Leaky Language Models: Stealing Architecture and Inference Optimizations via Per-Token Timing"
author: "Sadegh Majidi, Niloofar Mireshghallah, Kazem Taram"
date_fetched: 2026-07-25
date_published: 2026-07-22
topics:
  - ai-research-and-models
---

Majidi et al. (Purdue / CMU) show that fine-grained per-token generation timing — observable through any standard streaming API — leaks both deployment optimizations and architectural details of remote LLMs. No privileged access is needed; only timestamps between streamed tokens.

Two attacks are presented. The first detects **speculative decoding** by exploiting the fact that draft models have shorter context windows than the main model. By gradually increasing prompt length in a number-memory task and watching for a sudden latency spike (where the draft model starts disagreeing because it can't see far enough back), the attacker confirms speculative decoding and pinpoints the draft model's context length. Against real APIs, this revealed that Google Gemini Flash 2.5 uses speculative decoding with a ~128K-token draft context, and Gemini Flash 1.5 with a ~32K draft context.

The second attack recovers **model architecture**: number of layers (L), hidden dimension (H), and attention heads (A). The authors build a hybrid analytical–empirical timing model — theoretical asymptotic terms corrected by linear regression on instrumented open-source models — then use it as an oracle to search over plausible (L, H, A) configurations. A two-phase ensemble (classifier + per-kernel regressors) handles cuBLAS's dynamic kernel selection. On Llama 3.2 3B, the near-correct architecture appears in the top-5 guesses over 65% of the time for (H, L) jointly; individual parameters are often recovered at higher rates. The predictor generalizes across model families (Qwen, Phi, Gemma) and was demonstrated against a remote Llama 3.1 8B API, ranking the correct hidden dimension #1 and the near-correct full config #8 out of 2,068 candidates.

Mitigation is hard: constant per-token timing would multiply latency; token buffering degrades user experience. The authors frame this as an open problem — defending against timing side channels without sacrificing the low-latency streaming that makes them observable in the first place.

---
*Sources: [[raw/leaky-language-models]]*
*Last updated: 2026-08-01*
