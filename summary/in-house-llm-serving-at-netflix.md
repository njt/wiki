---
url: https://netflixtechblog.com/in-house-llm-serving-at-netflix-a5a8e799ea2c
title: "In-House LLM Serving at Netflix"
author: Netflix Technology Blog (AI Platform's Model Runtime team and Inference team)
date_fetched: 2026-07-21
date_published: 2026-07
---

Netflix runs its full LLM stack in-house rather than consuming hosted APIs. The post walks through the key architectural decisions where alternatives were seriously considered: engine choice, model packaging, API design, deployment strategy, and constrained decoding at scale.

The GPU inference path delegates to a Model Scoring Service (MSS) built on NVIDIA Triton Inference Server with a Java control plane. Netflix selected vLLM as its paved-path engine over TensorRT-LLM, citing faster iteration (no multi-step compilation), extensibility hooks for custom decoding, and better debuggability. Models are packaged via Triton's vLLM backend, which requires only a JSON config pointing to weights and tokenizer, with a fallback to the Python backend when custom preprocessing is needed.

Netflix exposes both gRPC (its standard scoring interface) and an OpenAI-compatible HTTP frontend, making it nearly seamless for apps to graduate from hosted to self-hosted models. It had to patch Triton's OpenAI frontend — it was silently dropping the `response_format` parameter needed for structured output.

For deployment, Netflix recommends embedding variable configurations directly into the model to enable Red-Black deploys — atomic traffic shifting with fast rollback — and reserves Versioned deployment (separate instances per model version) for unavoidable breaking changes, since it temporarily doubles GPU cost.

A deep-dive covers constrained decoding at scale. Netflix pushes output constraints inside the decode loop using vLLM's custom logits processor, with each constraint modeled as a state machine. The initial V0 implementation ran per-request on CPU and hit tail latencies under concurrency because the GIL serialized logits processing across the batch. The team rewrote the processor for vLLM V1's batch-level API, reimplementing the hot path in C++ with multi-threading, which kept logits processing time flat as batch size grew. They also hardened it against partial prefills and KV-cache preemption, two vLLM V1 behaviors that broke naive state tracking.

Operational notes cover model caching via Amazon FSx to avoid slow cold starts, a unified metrics endpoint that merges vLLM and Triton Prometheus metrics (Triton's bridge surfaced only 9 of 40+ vLLM metrics), and future directions including GPU-kernel logits processors and system-prompt compression.
