---
url: https://blog.alexellis.io/local-ai-is-not-opus/
title: "Local Qwen isn't a worse Opus, it's a different tool"
author: Alex Ellis
date_fetched: 2026-07-05
date_published: 2026-06-17
site: alexellis.io
tags: [llm, localai, agents, openfaas, qwen, llama.cpp, gpu, hardware]
---

# Local Qwen isn't a worse Opus, it's a different tool

**Author:** Alex Ellis
**Published:** 17 June 2026
**Source:** https://blog.alexellis.io/local-ai-is-not-opus/

---

## Summary

Alex Ellis, founder of OpenFaaS, argues that comparing local models like Qwen 27B to frontier models like Claude Opus is a category error — they serve fundamentally different purposes. Drawing from his experience running a bootstrapped infrastructure company, he details the specific, caveated value local models have produced: airgapped customer support diagnostics, revenue recovery from telemetry analysis, and cost-controlled codebase reading. The post traces his hardware journey from dual 3090s to a $12,000 RTX 6000 Pro Blackwell, documents the looping problem that makes local models unsafe for unsupervised long-horizon work, and argues the near-Opus framing misses the point: local models are a different tool entirely, suited for privacy-sensitive, bounded, fixed-cost workloads.

---

## Key Arguments

### Three Drivers for Local AI

1. **Cost:** Coding plans are "clearly subsidised." GitHub Copilot's token-based pricing shift and Uber's $1,500/month/developer AI spend cap (~12% of median salary) signal that current pricing is unsustainable.

2. **Sovereignty & Privacy:** OpenFaaS products are built around these principles. Anthropic removing "Fable 5 model overnight" exemplifies serious vendor risk — "many of us are addicted to the source."

3. **Vendor Independence:** Local models answer the question "What if the frontier labs do X?"

### Benchmark Critique

Qwen 3.6 27B scores 77.2 on SWE-Bench Verified versus Opus 4.8's 88.6% — a gap Ellis calls misleading. Benchmarks are "a moving target" and SWE-Bench focuses on Python, whereas his team "write distributed systems in Go, where channels, contexts, and structs span across a large execution domain."

### The Tempering Blade Analogy

Ellis compares local models to heat-treating hand tools: "The model is running so hot, that it shoots past the goal and starts looping." He'd "never leave a blade tempering unattended, just like I'd never leave Qwen 3.6 27B working on a long horizon task."

### The Looping Problem

A vivid example: asking what commands to add to `faas-cli` resulted in the model repeating the same 5 suggestions in a loop — entries 58 through 72 in the output, identical — burning "600W of my electricity for a good half an hour." The model was "stuck, at the edge of its ability" unwilling to ask for help.

### Hardware Journey

- **3090 Era (2023 onward):** Started with one, needed two. Qwen 3.5 was the first usable model. Early failure: model told to explore a machine comprehensively "started reading every single file on my machine one by one, filled its context, then hallucinated the filenames." Scoping tightly gave usable results at ~40–50 tok/s. "Bad things start happening at Q4_0 on the keys part of the KV cache." One card required A/C power cycling.

- **RTX 6000 Pro Blackwell (~$12,000, now ~$15,400):** 96GB VRAM. Paid off but "not because it replaces our Claude subscriptions — it can't do that."

### Current Setup

Two independent llama.cpp instances serve Qwen 3.6 27B and Qwopus fine-tunes. UD-Q8_K_XL quantization, 262K context, speculative decoding via MTP achieving ~93% acceptance rate, boosting speed from 67 tok/s to 130–200 tok/s sustained. "Feels faster than using a cloud model."

### Specific Business Value

1. **Customer Support:** CLI tool "diag" captures OpenFaaS installation snapshots. Operators email dumps, which Ellis runs through an airgapped local model in an ephemeral VM — avoids leaking customer data to cloud services.

2. **Revenue Recovery:** Feeding telemetry into local models revealed a customer "had been under-reporting licenses and under-paying by about 4-5x for over 12 months." That alone paid for the RTX 6000 Pro. Earlier models "failed at arithmetic — 27.3K counted as 273,000" and once inferred churn incorrectly by ignoring usage frequency. His conclusion: "it's better to have them focus on analysis, not interpretation."

### Toilgate & Observability

Ellis built a provider for opencode called Toilgate to manage model access, routing, and identity. Two Shelly Plus Plugs monitor power — the RTX 6000 Pro pulls 600W during inference while "relatively quiet," compared to two 3090s at ~750W and "extremely noisy."

### What Local Models Are and Aren't Good For

**Good for:** Specialized, well-bounded tasks (customer support, maintenance, testing); codebase reading and explanation ("this is a superpower"); tasks requiring data privacy (telemetry, customer diagnostics); fixed-cost operations once hardware is purchased.

**Not good for:** Long-horizon, unsupervised agentic work; writing Go code all day ("their limited knowledge and attention shows up immediately in code review"); tasks requiring consistent adherence to brevity instructions.

### Concrete Recommendations

- Match model and harness to specialized tasks
- Use AGENTS.md files (detailed instructions improved CLI contributions significantly)
- Follow model card tuning notes on temperature, context, and quantization
- Experiment with fine-tunes like Qwopus
- Normalize running the same task with both local and cloud models
- Don't hand it long-horizon unsupervised work — "even our almost 15k USD card couldn't fix that"

### On Larger Models (70B+)

Ellis dismisses most 70B models as "genuinely old at this point, generations behind." Models like GLM 5.2, Kimi 2.7, or Deepseek V4 Flash require "4-6 RTX 6000 Pro cards" to even load quantized, putting them out of scope.

### Conclusion

Local Qwen is "of value for certain tasks and workflows" and "incredibly early" — it can only improve. The "near-Opus level" framing is a category error: local models are a different tool entirely, suited for privacy-sensitive, bounded, fixed-cost workloads, not unsupervised frontier-level coding.

---

## Notable Quotes

> "Local Qwen isn't a worse Opus, it's a different tool."

> "I'd never leave a blade tempering unattended, just like I'd never leave Qwen 3.6 27B working on a long horizon task."

> "It's better to have them focus on analysis, not interpretation."

> "The model is running so hot, that it shoots past the goal and starts looping."

> "Bad things start happening at Q4_0 on the keys part of the KV cache."

> "Even our almost 15k USD card couldn't fix that."

> "Many of us are addicted to the source."

> "Codebase reading and explanation: this is a superpower."

> "Their limited knowledge and attention shows up immediately in code review."
