---
url: https://huggingface.co/shisa-ai/shisa-de-1
title: "Shisa DE-1"
author: Shisa AI
date_fetched: 2026-09-22
date_published: not stated
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

Shisa DE-1 (Decision Engine) is an open-weight Apache-2.0 "decision model" — 25.2B total / 3.8B active parameters of Gemma 4 mixture-of-experts (128 experts, 8 active + 1 shared, 256K context) fine-tuned from `google/gemma-4-26B-A4B-it`. It takes one state and a list of named questions and returns one typed answer per question: instead of generating answer text, each answer is read from the next-token distribution at the answer boundary, restricted to the option letters. The defining move is that the training loss *is* the deployment readout — softmax cross-entropy over the K option-letter rows at the answer boundary — so there is no gap between what the model was optimized on and what the serving contract consumes. Train accuracy rose 0.888 → 0.936 → 0.968 across three epochs on just 3,632 examples covering compaction, fraud detection, anti-slop, and safety guardrails.

The serving contract is one request per question: render the state, question, and option list with the model's chat template, call with `max_tokens=1` and `logprobs`, restrict the letter logits to the K valid option letters, and take the argmax (choice) or the normalized probability of the Yes row (`noul` questions). Tokens outside the option list are discarded even when they outrank valid options; the sampled text is "shown for debugging only — the contract reads the distribution, not the sample." Each option letter must be a single token, capping mutually exclusive options at 26 (the Banking77 suite with 77 options is reported as a partial run for this reason). Served on mainline vLLM across 2× NVIDIA H20-3e (TP2), the fraud-detect suite sustains 596–684 req/s from concurrency 64 to 512 with 12 ms p50 at concurrency 1.

Against a 29-suite harness (plus option-order-reversed variants of 13), DE-1 beats the hosted Jev baseline it replaces on fraud-detect (0.999 vs 0.982 AUROC), guardrails (0.917 vs 0.583 acc), and antislop (0.997 vs 0.970), at 20 ms p50 versus Jev's 223 ms hosted latency — while the base Gemma 4 with the same readout already scores 0.985/0.833/0.973 on those suites. Smaller specialist classifiers (576M GLiFormer, 568M BGE-M3, 150M ModernCE) trail badly on the decision suites, supporting the card's thesis that larger LLMs generalize better and still fit real-time windows. A trained dot-product head over frozen hidden states scored lower on the sealed compaction test (0.837 vs 0.8902 AUROC), an ablation arguing the LM readout itself is load-bearing.

The card is unusually honest about limits: held-out decision families are unsolved (0.491 vs Jev's 0.810 on kev-transfer-v1, unmovable by prompt-scaffold A/B), arithmetic and counting probes fail at every model size, option order shifts answers (mean absolute delta 0.029; the 11-option counting probe picks count 3 on 22 ascending vs 40 descending presentations), and post-hoc temperature calibration is fitted per answer kind (T_noul 1.69, T_choice 1.90) with an instruction to refit on your own rows before treating `noul` probabilities as confidence. No generation quality is claimed: open-ended generation, chat, and tool use are unsupported by the evidence, and the vision and audio paths were not evaluated.
