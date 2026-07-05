# JetBrains Mellum2

JetBrains open-sourced Mellum2 on June 1, 2026: a 12B Mixture-of-Experts coding model (2.5B active per token) under Apache 2.0. It's a "focal model" — not a frontier general-purpose LLM, but a fast, specialized component for high-frequency software engineering tasks like routing, RAG, sub-agent orchestration, and air-gapped deployment. Trained from scratch on ~10.6T tokens with a three-phase curriculum, it ships in Base, Instruct, and Thinking variants. The thinking variant hits 78.4% on EvalPlus, beating Qwen3.5-9B and Seed-Coder-8B, while matching Qwen2.5-7B throughput at ~192 tok/s on a single H100.

---

## Key Quotes

> "Frontier models will continue to push the limits, but practical AI products also require focal models: fast, specialized components that handle high-frequency tasks efficiently."
> — Anton Semenkin and Nikita Pavlichenko, JetBrains AI Blog

This is the article's thesis and its most useful contribution: a named concept for something many practitioners already sense. Not every task needs a frontier model. The orchestration layer, the RAG summarizer, the routing classifier — these are high-frequency, latency-sensitive jobs where throughput matters more than reasoning depth. "Focal model" gives you vocabulary for the architectural decision to split your pipeline across model tiers instead of throwing Opus at everything. Cf. [[Smart Models Dumb Pipes]] for the complementary pattern — focal models are the "dumb pipes" made intelligent enough to route.

> "The future belongs to coordinated systems, not single models."

JetBrains is making the same bet as [[Step 3.7 Flash]]'s Advisor Mode and the [[Components of a Coding Agent]] harness-first philosophy: the unit of analysis isn't the model, it's the system. A single frontier model is a bottleneck; a coordinated system of specialized models is an architecture. Mellum2 is designed to be a component in that system, not the star.

> "Open source is how better tools get made."

Simple, unpretentious, correct. No manifesto, no grand strategy document — just the quiet confidence of a company that's been shipping developer tools for 25 years. Compare to Meta's elaborate Llama openness framing or Google's "open but not open source" positioning. JetBrains ships Apache 2.0 and moves on.

---

## Architecture Deep Dive

**MoE with 2.5B active parameters.** 12B total, 64 experts, 8 active per token. This is the key number: you get 12B worth of knowledge capacity but 2.5B worth of per-token inference cost. For a focal model doing high-throughput work, that's the right tradeoff. The deployment complexity of MoE (expert loading, routing failures, uneven utilization) is real, but JetBrains clearly thinks the throughput win justifies it. [[Granite 4.1]] made the opposite bet — dense architecture, predictable latency, no routing overhead. Two credible teams, two different answers to the same question.

**Multi-Token Prediction as dual-use infrastructure.** The MTP head serves as both an auxiliary pre-training objective AND a built-in draft model for speculative decoding. This is elegant: you're already training the draft model during pre-training, so speculative decoding at inference time is essentially free. Most models bolt speculative decoding on as a separate component; Mellum2 builds it into the architecture.

**Sliding window on 3 of every 4 layers.** Combined with grouped-query attention (4 KV heads), the memory footprint stays small while the 131K context window stays large. No RULER or needle-in-haystack scores provided, though — the long-context claims are unverified. This is a pattern with vendor model announcements: big context window numbers, no degradation charts. See [[Benchmark Exploitation]].

**Not multimodal by design.** Text and code only. This is refreshing — JetBrains explicitly says specialization "ensures the model excels in software engineering environments while remaining lean and fast." Most model vendors chase multimodality as a checkbox feature; JetBrains treats it as a tradeoff they declined to make.

---

## Key Themes

- **#concept Focal models** — The named idea: frontier models push capability boundaries; focal models handle high-frequency production work. A useful taxonomy for system design conversations. Complements the "smart models, dumb pipes" pattern and the harness-first philosophy of [[Components of a Coding Agent]].

- **#tool MoE for throughput** — 12B total / 2.5B active is the architectural bet. You pay for knowledge capacity at training time, you pay for inference at the active parameter count. For a model designed to be called hundreds of times per second in an orchestration pipeline, this is the right axis to optimize.

- **#pattern Speculative decoding as architecture, not afterthought** — The MTP head trains a draft model during pre-training, then reuses it for speculative decoding at inference. This is good engineering: don't bolt on what you can build in. Compare to the [[Honey I Shrunk the Coding Agent]] approach of adapting the scaffold around the model — both are about designing the system holistically rather than optimizing components in isolation.

- **#pattern Apache 2.0 as competitive strategy** — No restrictions, no acceptable use policy, no "you can't use this to compete with us" clause. JetBrains commoditizes the model to differentiate on the IDE and developer tooling. Same playbook as [[Granite 4.1]] (IBM) and opposite to Meta's Llama license. For regulated industries and air-gapped deployments, the license is the feature.

- **#tool Self-hosted coding agents** — Mellum2 is explicitly designed for private deployment: "keep your code and data fully under your control." This is the [[Personal Agents]] and [[Local and Open Source Inference]] use case — a capable coding model that runs on your hardware, behind your firewall, with no API calls to third parties.

---

## Critical Analysis

**"Focal model" is a genuinely useful concept.** The term itself is the contribution. "Small model" implies worse; "specialized model" implies narrow. "Focal model" captures what's actually different: it's designed for a specific point in the system architecture, not as a general-purpose replacement for anything. Every agent orchestration discussion now has vocabulary for "this step needs a frontier model, this step needs a focal model." That's real intellectual progress from a vendor blog post.

**The benchmarks are self-reported and incomplete.** JetBrains shows Mellum2 beating Qwen3.5-9B and Seed-Coder-8B on EvalPlus, but these are their numbers from their eval harness. The same caution applies as with [[Granite 4.1]]: vendor benchmarks are marketing until independently reproduced. No LiveCodeBench or SWE-bench scores provided. No latency-at-scale data beyond "192 tok/s on one H100." For a model whose entire pitch is production throughput, the absence of detailed latency data under realistic concurrent load is notable.

**The "where Claude Code can't" framing is TNS editorializing, not JetBrains' positioning.** The blog post never mentions Claude Code. The actual positioning is about orchestration and private deployment — Mellum2 as a component in a pipeline, not as a replacement for frontier coding assistants. The TNS headline ("to go where Claude Code can't") is linkbait that distorts the actual story. Mellum2 doesn't compete with Claude Code; it competes with using Claude Code for tasks that don't need Claude Code.

**MoE is the right call for a focal model but complicates local deployment.** Running MoE models locally means managing expert loading and routing — more moving parts than a dense model. For the air-gapped deployment use case JetBrains pitches, this matters. A 2.5B-active MoE may actually be harder to deploy reliably than a 7B dense model, even if the throughput numbers are better on paper. [[Datacenter GPU in a Gaming PC]] shows what local inference actually looks like — the gap between spec sheet and reality is where MoE complexity bites.

**The three variants solve different problems but create a selection problem.** Base, Instruct, and Thinking — which one do you use for which task? The blog post doesn't provide a decision matrix. For orchestration, you probably want Instruct (fast, direct). For complex code generation, Thinking (reasoning traces). For fine-tuning, Base. But the boundaries are fuzzy, and picking wrong means either wasted latency (Thinking for simple routing) or inadequate reasoning (Instruct for complex generation). A routing guide would make the model more usable.

**JetBrains' position is interesting and underappreciated.** They're neither a pure AI company (Anthropic, OpenAI) nor a pure infrastructure company (Hugging Face). They're an IDE company that now ships models. The strategy: open-source the model, differentiate on the developer experience. If Mellum2 becomes the default focal model in agent pipelines, JetBrains wins even if nobody pays for model inference — because the model drives adoption of JetBrains' AI tooling ecosystem. It's the [[Granite 4.1]] playbook (commoditize the model, sell the platform) executed by a company with 25 years of developer trust.

**What's actually new here vs. what's catching up.** MoE architectures for coding models aren't new ([[Step 3.7 Flash]], Qwen, DeepSeek). Multi-token prediction isn't new (Meta, DeepSeek). Apache 2.0 coding models aren't new ([[Granite 4.1]], StarCoder). What's new is the combination — a purpose-built focal model from a company that actually ships developer tools, with a clear architectural philosophy and no licensing friction. The innovation isn't in any single technical choice; it's in the product thinking about what a model is *for*.

---

## Related

- [[Granite 4.1]] — IBM's Apache 2.0 dense model family: opposite architectural bet (dense vs. MoE), same licensing strategy
- [[Step 3.7 Flash]] — Another MoE coding model (196B/11B active) with Advisor Mode for cost efficiency
- [[Honey I Shrunk the Coding Agent]] — The scaffold matters more than the model; Mellum2 as focal model is scaffold-aware design at the model level
- [[Components of a Coding Agent]] — Raschka's taxonomy: Mellum2 is designed as a component in the harness, not as the harness itself
- [[Smart Models Dumb Pipes]] — Focal models are the "dumb pipes" made smart enough to route and summarize
- [[Local and Open Source Inference]] — The self-hosted deployment use case Mellum2 is targeting
- [[Personal Agents]] — Private deployment for personal coding agents
- [[Self-Hosted LLMs]] — Hardware-to-model mapping for local inference
- [[Datacenter GPU in a Gaming PC]] — What local MoE inference actually looks like on real hardware
- [[2025 in LLMs]] — Simon Willison's landscape survey; focal models as a category were emergent in 2025
- [[Recent Developments in LLM Architectures]] — MoE, KV sharing, speculative decoding trends that Mellum2 participates in
- [[Notes from the AI Now Summit by Mistral]] — The "model alone isn't enough" thesis that JetBrains is betting on
- [[Benchmark Exploitation]] — Why vendor-reported benchmarks need independent verification

---

*Primary source: [JetBrains AI Blog](https://blog.jetbrains.com/ai/2026/06/mellum2-goes-open-source-a-fast-model-for-ai-workflows/) (Anton Semenkin, Nikita Pavlichenko, June 1, 2026)*
*Also referenced: [The New Stack](https://thenewstack.io/jetbrains-mellum2-open-source-coding-model/) (Paul Sawers, June 1, 2026) — article body not retrievable due to paywall*
*arXiv technical report: [2605.31268](https://arxiv.org/abs/2605.31268)*
*Raw source: [[summary/mellum2]]*
*Last updated: 2026-06-04*
