# In-House LLM Serving at Netflix

Netflix's AI Platform team details their production LLM serving stack: vLLM inside NVIDIA Triton, with a Java control plane, gRPC + OpenAI-compatible HTTP frontends, and a custom constrained-decoding system rewritten from per-request Python to batch-level C++ when vLLM V1 landed. The article is refreshingly honest about the gaps between vendor tooling and production reality — silently dropped API parameters, metrics endpoints that miss 75% of what you need, and deployment strategies that break when I/O schemas change.

---

## Key Quotes

> "We expose the OpenAI-compatible API as an additional frontend alongside gRPC" because "graduating from a hosted to a self-hosted model is nearly seamless with the ecosystem frontend."

This is the operational thesis in one sentence. Netflix isn't optimizing for theoretical purity — they're optimizing for the developer experience of teams moving from OpenAI's API to internal infrastructure. The gRPC path handles the heavy production traffic; the OpenAI-compatible path handles developer onboarding. Both share the same backend, so there's no quality cliff between prototype and production. This is the [[Smart Models Dumb Pipes]] pattern applied to API design: smart models behind dumb (standardized, interchangeable) interfaces.

> "Triton's built-in bridge surfaces only 9 of 40+ vLLM metrics" — missing token throughput, KV cache utilization, and prefix cache hit rates.

The observability gap as architectural critique. Triton wraps vLLM but abstracts away the metrics that actually matter for [[KV Cache Locality]] and capacity planning. Netflix's fix — a lightweight HTTP proxy merging both endpoints — is 50 lines of operational glue that should have been upstream. The broader lesson: when you wrap one open-source project inside another, the seams are where observability dies. See also [[Inference Cost Napkin Math]] for why those missing metrics (especially KV cache hit rates) are the ones that determine your margin.

> "CPU time in logit processing therefore grows linearly with batch size, hitting tail latencies."

The GIL strikes again — and the bottleneck was invisible in single-request benchmarks. This is the diagnostic pattern that separates production engineers from model researchers: knowing that performance regressions hide in concurrency, not in unit throughput. The fix (batch-level C++ with multi-threading) is the same pattern as vLLM's own evolution from V0 to V1: move from per-request Python to batch-level compiled code. Netflix just had to do it for their custom logic that vLLM doesn't ship.

> "Embedding variable configurations directly into the inference model" lets you use Red-Black deployment instead of Versioned deployment, avoiding the GPU-cost spike of running two full model versions simultaneously.

The cleanest deployment insight in the article, and the one most applicable beyond Netflix. Schema changes are the enemy of zero-downtime deploys. If your model's I/O contract is stable (because config lives inside the model, not in external orchestration), you can use cheap atomic traffic shifts. If config is external, every change is a breaking change and you pay the Versioned tax. This is the deployment equivalent of the [[Layer-First Pattern — Keep Data Out of the LLM Context]]: keep variable state inside the model boundary, not outside it.

> "We git-subtreed and patched the frontend" after discovering Triton's OpenAI-compatible frontend "silently dropped the response_format parameter."

Silent parameter dropping is the worst kind of integration bug — it doesn't fail, it just produces wrong output that looks right. The fact that Triton's frontend silently ignored a parameter that Netflix's constrained decoding depended on is a reminder that the OpenAI API compatibility surface is wide but shallow. Everyone implements the easy parts; the hard parts (like `response_format` with JSON schema) get quietly skipped. Netflix's solution — `git subtree` the frontend and maintain a fork — is ugly but honest. The alternative is waiting for upstream to prioritize your use case.

---

## Key Themes

- #tool **vLLM as production inference engine** — Netflix's selection criteria (custom architecture support, extensibility hooks, debuggability) are a useful checklist for anyone evaluating inference engines. TensorRT-LLM lost on flexibility, not performance. The gap closed, and flexibility won.
- #pattern **Dual-frontend API design** — gRPC for internal production traffic, OpenAI-compatible HTTP for developer onboarding. Both hit the same backend. This is the right pattern for any org running self-hosted LLMs: don't make developers learn your internal protocols.
- #pattern **Versioned vs. Red-Black deployment for models** — Two strategies with a clean decision boundary: if your model I/O schema is stable, use Red-Black (cheap, atomic). If schemas change, use Versioned (expensive, decoupled). The insight is that embedding config in the model moves you from Versioned to Red-Black territory.
- #concept **Constrained decoding at batch scale** — The architectural jump from per-request logits processing (V0, GIL-bound, linear latency growth) to batch-level C++ (V1, multi-threaded, flat latency). The operational hardening for partial prefills and preemption is the kind of detail you only discover in production.
- #concept **Observability at integration seams** — When you compose open-source components (Triton + vLLM), the metrics you need live at the boundary between them. Netflix's merged `/metrics` endpoint is a pattern worth stealing. If you can't see KV cache hit rates, you can't tune [[KV Cache Locality]].
- #tool **Triton Inference Server** — NVIDIA's model server as the substrate, with vLLM as the backend engine. The vLLM backend (JSON config → dynamic specs) is dramatically simpler than the Python backend (explicit I/O tensors at packaging time), but version mismatches between Triton and vLLM create brittle failure modes.

---

## Critical Analysis

**The article is what a production ML infrastructure post should be: specific failures, not aspirational architecture.** Too many engineering blogs describe what they built; this one describes what broke and how they fixed it. The silent `response_format` drop, the GIL-bound logits processor, the metrics gap, the partial-prefill tracking bug, the preemption state-machine reset — these are the details that separate a real production system from a conference talk.

**The vLLM-over-TensorRT-LLM decision is a leading indicator of where inference infrastructure is heading.** Netflix made the call in summer 2025, and the criteria they used (flexibility, debuggability, practitioner familiarity) have only become more decisive since. Compiled engines optimize for the steady state; open-source engines optimize for the rate of change. When the model landscape is shifting every quarter, rate of change beats steady-state efficiency. The same dynamic that made PyTorch eat TensorFlow is now playing out in inference engines.

**The constrained decoding deep-dive is the article's most valuable section, and it's not close.** The batch-level rewrite, the C++ hot path, the `update_state` API complexity, the preemption handling — this is a graduate seminar in what it actually takes to run structured output at scale. The vLLM V0 → V1 migration as experienced by a platform team (not the vLLM authors) is the kind of war story that saves other teams months of debugging. The future direction — "vectorized logits processors that run as fused GPU kernels instead of CPU code" — suggests this is still an unsolved problem at Netflix scale.

**What's missing: cost.** The article is detailed about architecture and operations but silent on the economics. What does this stack cost per token? Per model? How does it compare to hosted APIs for Netflix's workload mix? The omission is conspicuous given that cost is the primary argument for self-hosting. Even a ballpark would make the article stronger — without it, "we run our own LLM stack" reads as an engineering flex rather than a business decision.

**The deployment strategy section solves a real problem but could go further.** Embedding config in the model to avoid schema changes is clever, but the deeper insight — that deployment strategy is downstream of API design — goes unstated. If you design your model interfaces to be extensible (optional fields, versioned schemas), you can stay on Red-Black forever. The article treats schema changes as unavoidable; a more opinionated take would argue they're a design smell.

---

*Sources: [[raw/in-house-llm-serving-at-netflix]]*
*Last updated: 2026-07-21*
