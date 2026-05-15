# Granite 4.1

IBM's open-source (Apache 2.0) model family in 3B, 8B, and 30B sizes, released April 2026. All dense decoder-only transformers with identical training pipelines, a 4.1M-sample curated fine-tuning dataset, and four-stage RL that caught and corrected a chat-training regression in math performance. The 8B dense model matches the prior 32B MoE across nearly every benchmark. Available on Ollama, Hugging Face, vLLM, and IBM's API.

---

## Key Quotes

> "The only difference between them is size."

IBM chose architectural uniformity over the MoE tricks everyone else is using. This makes Granite 4.1 boring in the best way — predictable latency, no routing overhead, no specialist layers to debug. In production, boring is the feature.

> "A deliberately curated 4.1 million."

The fine-tuning dataset size. Compare this to the web-scale scrapes that feed most models. IBM ran every assistant response through an LLM-as-Judge filter across six dimensions (instruction following, correctness, completeness, conciseness, naturalness, calibration) and cut everything below threshold. Automatic rejection for hallucinations, false premises, and incorrect computations regardless of score. This is the quality pipeline that most model vendors hint at but don't document.

> "What IBM built here is a production-first model family from a team that clearly spent more time fixing problems than announcing them."

The thesis of the review. IBM doesn't have the AI brand cachet of Meta or Google, but the engineering discipline — catching the RLHF math regression, the staged context window extension with model merging, the six-dimension data filter — reads like enterprise infrastructure thinking applied to model training.

## The RLHF Regression Story

This is the most instructive detail: Stage 1 joint-trained across nine domains. Stage 2 RLHF on chat improved AlpacaEval by ~19 points but **caused math scores to drop**. GSM8K and DeepMind-Math both regressed. Stage 4 was a dedicated math RL run to recover the damage — and it worked (+3.8 GSM8K, +23.5 DeepMind-Math).

IBM's willingness to document this is rare. Most model vendors bury training accidents. The joint-training-then-repair pattern is a lesson for anyone doing multi-objective RL: you can't just stack objectives and hope. You need to detect regressions and have a recovery strategy.

## Key Themes

- **Dense over MoE.** #concept The 8B dense model beats the prior 32B MoE (9B active). Dense architectures are simpler to deploy, predictably latented, and don't suffer from expert routing failures. For tool-calling agents where latency matters as much as capability, this matters.
- **Curated data over scale.** #concept 4.1M quality-filtered samples. LLM-as-Judge across six axes. Automatic hallucination rejection. The bet: better data beats more data when you're targeting enterprise reliability.
- **Tool calling as the differentiator.** #pattern BFCL V3 scores (68.3 for 8B, 73.7 for 30B) are strong for open models. This isn't about chat quality — it's about being useful as infrastructure. The [[Smart Models Dumb Pipes]] pattern needs models that can call tools reliably.
- **Production-first licensing.** #tool Apache 2.0 with no restrictions. Compare to Meta's Llama license or Gemma's terms. IBM is making a different bet: commoditize the model, sell the infrastructure and services around it.

## Critical Analysis

**The benchmark problem.** These are all self-reported scores from IBM's own evaluation harness. The author notes this, but it's worth emphasizing: independent benchmarks put most models lower than vendor-reported numbers. The 8B beating the old 32B MoE is the most important claim, and it needs third-party verification. See [[Benchmark Exploitation]] for why vendor benchmarks warrant skepticism.

**The 512K window is honest about degradation.** RULER scores dropping from 83.6 (32K) to 73.0 (128K) for the 8B isn't great, but it's honest. Most vendors claim huge context windows and bury the needle-drop charts. IBM's transparency here is a competitive signal in itself.

**IBM's AI credibility gap.** IBM has Watson baggage. Granite doesn't get the attention that Llama or Gemma or Qwen get, even when the engineering is arguably better. The review itself notes: this is a team that spent more time fixing problems than announcing them. That's the enterprise culture, but it means Granite flies under the radar of developers choosing models based on Twitter discourse.

**The tool-calling story is bigger than the benchmarks suggest.** A 68.3 BFCL V3 at 8B parameters, running on consumer hardware via Ollama, with Apache 2.0 licensing — that's directly relevant to anyone building [[Personal Agents]] or local agent infrastructure. [[2025 in LLMs]] noted that reliable tool-calling remained cloud-only in 2025. Granite 4.1 is evidence that gap is closing.

**The 3B is the sleeper.** 82.1 IFEval, 87.0 GSM8K, 60.8 BFCL V3 — at 3B parameters. That's edge-deployment territory: on-device agents, embedded tool calling, local RAG pipelines. If these numbers hold up on independent evals, the 3B could be the most impactful model in the family for agent infrastructure. Compare to [[MimiClaw]] running on an ESP32 — the 3B is too big for microcontrollers but perfectly sized for laptops and phones.

**What's missing.** No reasoning benchmarks (ARC, AGIEval). No multilingual eval despite IBM's enterprise customer base. No latency or throughput data. No comparison against Claude or GPT on tool calling — BFCL is an internal IBM benchmark. And the article doesn't touch on what the open-source community actually does with these models once they're on Hugging Face.

---

*Sources: [[raw/granite-4-1-ibm-open-source-model-family]]*
*Last updated: 2026-05-15*
