---
url: https://venturebeat.com/technology/cohere-open-sources-a-coding-agent-that-runs-on-a-single-h100
title: "Cohere Open-Sources North Mini Code: A Coding Agent That Runs on a Single H100"
author: VentureBeat (article behind paywall; sourced from Cohere blog post "Introducing North Mini Code" dated 2026-06-09)
date_fetched: 2026-06-15
date_published: 2026-06-09
topics:
  - ai-research-and-models
---

## Source Notes

The VentureBeat article returned HTTP 403 (paywall/anti-bot). Content reconstructed from:

1. Cohere's official blog: "Introducing North Mini Code: Cohere's first model for developers" (2026-06-09) — fetched successfully via WebFetch
2. Web search results confirming technical specifications from multiple outlets (DevOps.com, The Block Beats, KuCoin News)

## Article Summary (from Cohere blog and secondary sources)

Cohere released **North Mini Code (North-Mini-Code-1.0)**, its first open-source agentic coding model. The model uses a Mixture-of-Experts architecture with 30B total parameters but only 3B active per forward pass, allowing it to run on a single NVIDIA H100 GPU at FP8 precision. Licensed under Apache 2.0, it's available on Hugging Face, Cohere API, Model Vault, OpenRouter, and OpenCode.

### Technical Specifications

| Spec | Detail |
|---|---|
| Model name | North-Mini-Code-1.0 |
| Architecture | Mixture-of-Experts (MoE) |
| Total parameters | 30B |
| Active parameters | 3B |
| Context length | 256K total; 64K max generation |
| License | Apache 2.0 |
| Min. hardware | 1× H100 at FP8 |
| Availability | Hugging Face (CohereLabs/North-Mini-Code-1.0), Cohere API, Model Vault, OpenRouter, OpenCode |

### Benchmark Performance

- **Artificial Analysis Coding Index**: 33.4 — above GLM-4.7-Flash (25.9) and close to Qwen3.6 35B (35.2)
- **Output throughput**: 2.8× faster than Mistral's Devstral Small 2 under same hardware
- **Inter-token latency**: 30% lower than Devstral Small 2
- **Inference speed**: ~199–208 tokens/second on Cohere API
- **Evaluation**: SWE-agent harness for SWE-Bench Verified/Pro; simple ReAct harness for Terminal Bench v2

### Specialization

The model is specifically trained for agentic coding workflows:
- Coordinating sub-agents
- Code generation and code reviews
- System architecture design
- Software engineering agentic tasks

### Trade-offs

- Strong on coding benchmarks
- Weaker on non-coding agentic tasks (14% on GDPval-AA, 37% on τ²-Bench Telecom)
- Known to be "chatty" / verbose in output

### Strategic Context

This is Cohere's first model in what they describe as a "new generation of powerful models." The Apache 2.0 license marks a significant shift from Cohere's previous CC-BY-NC licensing, positioning North Mini Code for the "sovereign developer ecosystem" — developers who want to run their own models on their own hardware.
