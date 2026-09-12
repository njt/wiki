# 0xSero

AI coding agent practitioner operating at the intersection of model optimization, data extraction, and long-running autonomous workflows. Runs Sybil Solutions ("Building production systems with AI"). Active on HuggingFace publishing REAP-pruned expert models calibrated specifically for code generation and tool calling.

---

## What They Build

### REAP-Pruned Models

Publishes expert-pruned MoE models on HuggingFace using Cerebras' REAP (Router-weighted Expert Activation Pruning) methodology. Each model is calibrated on three datasets: `evol-codealpaca-v1` (code generation), `xlam-function-calling-60k` (tool calling), and `SWE-smith-trajectories` (agentic multi-turn coding). The calibration target is telling: they're optimizing not for benchmark scores but for the specific distribution of agentic coding work.

Models include GLM-4.7-REAP-45 (45% pruned, 197B active), Qwen3.5-264B-REAP, and gemma-4-21b-a4b-it-REAP — some with INT4 quantization bringing a 358B model down to ~108GB.

### ai-data-extraction Toolkit

A zero-dependency Python toolkit (680 stars) that pulls complete conversation history from eight AI coding tools: Claude Code, Codex, Cursor, Windsurf, Trae, Continue, Gemini CLI, and OpenCode. Outputs standardized JSONL with messages, code context, diffs, tool use, and metadata. Built for ML training pipelines — the README includes Unsloth fine-tuning examples.

This is the unglamorous plumbing of the "train your own coding model" pipeline. Before you can fine-tune, you need clean data from your actual workflow. Most people talk about training; 0xSero built the extraction layer.

### Parchi

Chrome extension supporting BYOK (bring your own key) workflows for very long-running autonomous agent tasks. Cited in AINews' March 2026 agent harness roundup alongside the broader convergence on parallel, long-running agent stacks. Integrates with their pruned models (Qwen3.5-REAP).

---

## Key Themes

- **#tool** — ai-data-extraction fills a real gap: nobody else built a unified extractor across the fragmented AI coding tool landscape
- **#pattern** — Model optimization calibrated to agentic workloads, not generic benchmarks. The calibration dataset IS the product spec
- **#concept** — BYOK workflows: run autonomous agents on your own keys, your own hardware, your own models. The anti-SaaS posture applied to agents
- **#person** — Practitioner, not theorist. Builds the plumbing, publishes the models, teaches live

---

## Critical Analysis

**What's interesting:** 0xSero represents a specific archetype that's emerging — the agent infrastructure practitioner who works below the framework layer. They're not building Yet Another Agent Harness; they're optimizing the models that agents run on and building the data pipelines that feed those optimizations. This is the "pick and shovel" layer of the agent economy.

**What's undersold:** The ai-data-extraction toolkit is more significant than its 680 stars suggest. Training data extraction from proprietary coding tools is a hard problem that most teams solve with bespoke scripts. A standardized, multi-tool extractor is infrastructure. The fact that it's MIT-licensed and zero-dependency suggests it's meant to be absorbed into other projects rather than monetized directly.

**The REAP calibration choice is telling.** Most model publishers optimize for HumanEval or MBPP. 0xSero calibrates on SWE-smith-trajectories — multi-turn agentic coding sessions. This is a bet that agentic coding has a different token distribution than single-turn code completion, and that models should be optimized accordingly. It's the right bet, but it also means these models may underperform on standard benchmarks, which creates a discoverability problem.

**The homelab matters.** The 8x 3090 rig blog post signals a commitment to local inference that goes beyond cost savings. If you're extracting training data from your own coding sessions and fine-tuning models on your own hardware, you've closed the loop. No external API sees your code, your conversations, or your models. This is the privacy-maximalist position made practical.

---

## Connections

- [[Local and Open Source Inference]] — The 8x 3090 homelab is a concrete instance of the local inference thesis
- [[Personal Agents]] — Parchi's BYOK model is the personal agent philosophy applied to autonomous workflows
- [[Agent Memory and Context]] — ai-data-extraction treats coding tool history as training data; connects to the memory-as-training-data idea
- [[Scaling Long-Running Agents]] — Parchi tackles the long-running autonomous agent problem from the harness side
- [[Agent Orchestration]] — BYOK workflows intersect with multi-agent coordination patterns
- [[Three Tier Memory]] — Extracting conversation history for training is the cold tier of the memory pipeline
- [[Building low-level software with only coding agents]] — Similar practitioner profile: shipping real infrastructure, not demos
- [[claude-ctrl]] — "An instruction in context is not a constraint" — REAP calibration is the model-level equivalent

---

*Sources: [[summary/0xSero-tweet-2050607389498372588]]*
*Last updated: 2026-05-15*
