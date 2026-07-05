# Granite Libraries and Project Granite Switch

IBM's push to bring software engineering modularity to LLMs: adapter functions (LoRAs/aLoRAs) that slot into base models like library imports, and a switching layer that dynamically flips between them at inference time without losing KV cache. Part of the broader "generative computing" vision — treat models as composable, not monolithic. All Apache 2.0.

---

## Key Quotes

> "Models are just code with data, just a lot more data than code. We haven't learned the lessons of software for LLMs — we can build pieces separately."

Luis Lastras, IBM Research director of language and multimodal models. This is the thesis statement. LLMs today are undifferentiated masses of capability — you can't swap out the safety module without retraining the whole thing. IBM is arguing we're at roughly the "spaghetti code" phase of AI engineering, and modularity is the next step up.

> The requirement checker takes a model response and a set of constraints, and returns whether the constraints are satisfied. When Granite 4.1 3B is explicitly prompted to do this, it achieves 51% balanced accuracy on IFEval. If the same model is equipped with the new Granite Library requirement-check adapter function, its accuracy jumps to 84%.

This is the number that matters. A 33-point accuracy jump from a single adapter on a 3B model. Prompting alone caps out at coin-flip territory; the adapter makes the model genuinely reliable at a narrow task. This is the difference between "we told it to check requirements" and "it actually checks requirements."

> Instead of needing to spin up a brand-new AI model for every different task, Granite Switch dynamically flips the right adapter on and off exactly when it's needed.

The switching layer preserves KV cache across adapter changes. Standard LoRA switching forces a cache flush — the model has to re-read the context from scratch at each step. aLoRA (activated LoRA) carries memory forward. In multi-step RAG pipelines, this is the difference between a pipeline and a stutter.

## Key Themes

- **Adapter functions as software libraries.** #concept The core innovation: small, independently trained modules with defined inputs and outputs that slot into a base model. They don't generate open-ended text — they perform a specific function (scoring relevance, detecting hallucination, checking safety). This is the API boundary that LLMs have been missing.
- **Generative computing.** #concept IBM's term for treating model inference as a deterministic programming paradigm, not a stochastic text generator. Mellea enforces this by wrapping adapter calls in typed Python functions with real-time formatting enforcement. The vision: `result = model.requirement_check(response, constraints)` as a reliable function call, not a prompt you hope works.
- **KV-cache-preserving adapter switching.** #pattern The technical breakthrough in Granite Switch. Standard LoRA switching invalidates the KV cache; aLoRA preserves it via an extra transformer layer and control tokens. This is the kind of systems-level optimization that makes the difference between a demo and a production pipeline.
- **Three libraries, three concerns.** #tool RAG Library (query rewriting, hallucination detection, citation generation), Core Library (requirement checks, certainty scoring, attribution), Guardian Library (in-line safety, factuality, policy checks). Each independently trained, independently adoptable. The Guardian Library is particularly interesting — it bakes safety into the model rather than bolting on a separate guardrail model, which is the current industry default.
- **Small models + adapters beat large generalist models.** #pattern The 3B Granite with adapters outperforms the base 3B by 33 points on IFEval. Combine this with Granite 4.1's 8B dense beating the prior 32B MoE, and the strategy becomes clear: small, cheap, fast models with swappable expertise modules are more practical than one giant model that's mediocre at everything.

## Critical Analysis

**The right problem, finally.** IBM is attacking the actual bottleneck in enterprise AI adoption, which isn't model capability — it's predictability and maintainability. Open-weight models are now good enough for most tasks. What's missing is the ability to *reason about them* the way you reason about software: this module does X, that module does Y, and when X breaks you fix X without touching Y. IBM naming this "generative computing" is a branding move, but the underlying insight is correct: LLMs need interfaces.

**The IBM credibility problem cuts both ways.** IBM has Watson baggage. Developers don't look to IBM for AI innovation, they look to Meta, Google, Anthropic, and the open-source collective. But IBM's enterprise DNA means they actually understand what "production" means in a way most model vendors don't. The willingness to document training accidents (see [[Granite 4.1]]) and build adapter infrastructure rather than just bigger models suggests a team that's thinking about the right problems, even if few people are paying attention.

**aLoRA is the sleeper innovation here.** Most of the attention will go to Granite Libraries because "adapter functions as libraries" is the easy-to-understand pitch. But the switching layer that preserves KV cache across adapter changes is the part that makes this actually work in production. Without it, multi-step pipelines pay a re-reading tax at every adapter boundary. With it, you get fluid composition. This is the kind of detail that separates shipping code from shipping products.

**The IFEval number needs scrutiny.** 51% → 84% on a single benchmark is impressive but narrow. IBM doesn't report across the full adapter library on multiple benchmarks. The aLoRA vs LoRA race is visual and convincing, but it's from IBM's own telemetry. Every model vendor's self-reported benchmarks warrant skepticism (see [[Benchmark Exploitation]]). That said, the mechanism — training a narrow expert adapter rather than prompting a generalist — is sound. The question is how well it generalizes to real-world task distributions, not benchmark datasets.

**Composable AI is an architecture bet, not a model bet.** This isn't about building a better model. It's about building better infrastructure around models. The adapter approach means you invest in training small, specialized components rather than ever-larger generalist models. If the cost curves for inference keep dropping (they will), the economic argument for this approach weakens — why bother with adapters when you can just throw a bigger model at it? But the reliability argument strengthens regardless of cost: a typed function that fails deterministically is better than a prompt that fails stochastically.

**What's missing.** No latency/throughput comparison against prompt-based approaches (just the LoRA vs aLoRA race). No multilingual adapter evaluation despite Granite 4.1 supporting 12 languages. No third-party validation of any claims. And critically: no discussion of adapter *composition* — can you stack a safety adapter and a RAG adapter and a requirement checker simultaneously? The switching layer architecture suggests one-at-a-time, which would be a significant limitation for real pipelines.

---

*Sources: [[summary/granite-libraries-project-switch]]*
*Last updated: 2026-06-09*
