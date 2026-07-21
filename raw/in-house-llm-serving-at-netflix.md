---
url: https://netflixtechblog.com/in-house-llm-serving-at-netflix-a5a8e799ea2c
title: In-House LLM Serving at Netflix
author: Netflix Technology Blog (AI Platform's Model Runtime team and Inference team)
date_fetched: 2026-07-21
date_published: 2026-07
---

# In-House LLM Serving at Netflix

By Netflix Technology Blog (AI Platform's Model Runtime team and Inference team)

Netflix chose to run the full LLM stack internally rather than consume hosted APIs. The piece focuses on key architectural decisions where alternatives were seriously considered — engine choice, model packaging, API design, deployment strategy, and output constraint enforcement.

## Architecture Overview

Netflix's member-scale ML is served by a unified JVM-based system handling routing, A/B testing, candidate generation, feature fetching, inference, post-processing, and logging. Two paths reach inference: gRPC through the serving system and a direct HTTP path for newer LLM-driven apps.

For GPU models, the serving system delegates inference to a remote service called **Model Scoring Service (MSS)**, which uses NVIDIA Triton Inference Server underneath. A Java control plane on top handles deployment, versioning, health checks, autoscaling, and multi-region rollout.

## Design Decisions and Implementation

### vLLM as the Paved-Path Engine

The platform originally used TensorRT-LLM. By summer 2025, open-source engines had largely closed the performance gap, and Netflix's workload mix had diversified. They "selected vLLM as our paved-path engine" based on:

- Loading custom architectures without multi-step compilation
- Extensibility hooks for custom decoding logic
- Better debuggability compared to compiled engines
- ML practitioner familiarity from research use

### Integrating vLLM into Triton

Triton offers two packaging approaches. The **Python backend** requires authors to define explicit I/O tensor specs at packaging time, creating tight coupling. The **vLLM backend** uses "just a JSON config pointing to the model weights and tokenizer" — specs are generated dynamically.

Two production issues emerged: version mismatches between Triton and vLLM (e.g., a removed module causing load failures), and the need for a fallback to the Python backend for models requiring custom preprocessing or non-standard execution.

### Ecosystem-Compatible HTTP Frontend

A key design goal was that LLM models should not be special snowflakes — every model is scored via the same gRPC call. However, they also "expose the OpenAI-compatible API as an additional frontend alongside gRPC" to interoperate with the broader LLM ecosystem. This makes graduating from hosted to self-hosted models nearly seamless.

They reused NVIDIA's Triton OpenAI-compatible frontend but discovered it silently dropped the `response_format` parameter. They "git-subtreed and patched the frontend" to properly translate it into vLLM's guided decoding parameters.

### Deployment Strategies

Two strategies:

- **Red-Black:** New version deploys alongside current; traffic shifts in phases with atomic rollback. Best when model interfaces are stable, but a coordination gap emerged when I/O schema changes are needed — old requests fail against new deployments during migration.
- **Versioned:** Maintains independent deployments per (modelId, modelVersion) pair. Multiple versions serve simultaneously, decoupling deployment from consumer updates. Trade-off is temporary GPU cost increase during transition.

Their recommendation: "embedding variable configurations directly into the inference model" to allow the cheaper Red-Black path, reserving Versioned for unavoidable breaking changes.

## Operational Notes

### Boot Sequence

- **Model caching:** Large LLMs are materialized on Amazon FSx at announcement time, avoiding slow S3/Hugging Face downloads at startup.
- **Embedded vs standalone Triton:** Configurable per-deployment — embedded when OpenAI API is needed, standalone otherwise.
- Other steps: extracting model package, installing custom vLLM plugins via Python entry_points, cleaning Prometheus multiprocess directory, gating gRPC until ready.

### Unified Metrics Endpoint

Observability gap: vLLM writes metrics to Prometheus multiprocess directory files, Triton has its own endpoint, and Triton's built-in bridge "surfaces only 9 of 40+ vLLM metrics" — missing token throughput, KV cache utilization, and prefix cache hit rates. They added a lightweight HTTP proxy merging both into a single `/metrics` endpoint.

## Deep-Dive: Constrained Decoding at Scale

Netflix pushes constraints inside the decode loop so outputs are compliant by construction, implemented via vLLM's custom logits processor interface with each constraint modeled as a state machine.

### Why the first implementation didn't scale

In vLLM V0, custom logits processors ran per-request on CPU. The GIL prevented parallelization, so "CPU time in logit processing therefore grows linearly with batch size, hitting tail latencies." The bottleneck was invisible in single-request benchmarks but surfaced under realistic concurrency.

### vLLM V1 enabled a batch-level design

V1 moved logits processing to batch level. Netflix rewrote the processor to operate on batch-level data structures, reimplementing the hot path in C++ with multi-threading. The V1 API requires explicit batch membership tracking via `update_state(batch_update)` — more complex but necessary for correct state in dynamically evolving batches. Logits processing time stayed flat as batch size grew.

### Operational Hardening

Two unanticipated issues:

- **Partial prefills:** V1's chunked prefilling meant a request could span multiple engine steps. BatchUpdate lacked granularity to distinguish full from partial prefills, so they added internal tracking.
- **Preemption:** Under memory pressure, vLLM may evict a partially completed request's KV cache and reschedule later with different prompt/output lists. They detect when token history shrinks between steps, reset the state machine, and reinitialize from the new prompt.

## Wrap Up

Future investments include: system prompt compression, asynchronous scheduling of vLLM V1, "Vectorized logits processors that run as fused GPU kernels instead of CPU code," and lower-precision model variants.

## Contributors

Key contributors: Liping Peng (model packaging workflow, Triton/vLLM integration); Hakan Baba, Nicolas Hortiguera, ZQ Zhang (GPU capacity, performance tuning, observability); Santino Ramos (vLLM for production, constrained decoding optimization); Binh Tang (initial custom model serving, framework benchmarking); Lanxi Huang and Daneo Zhang (serving development tools); Lingyi Liu (overall system architecture and technical decisions); Abhishek Agrawal and Shaojing Li (management leadership).

## Acknowledgements

The work leverages Triton, vLLM, PyTorch, and other open-source ML libraries. They also thanked partner teams in Netflix AI for Member Systems.
