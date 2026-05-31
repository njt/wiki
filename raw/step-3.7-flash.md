---
url: https://static.stepfun.com/blog/step-3.7-flash/
title: "Step 3.7 Flash — A high-efficiency Flash model for Real-World"
author: StepFun
date_fetched: 2026-05-31
date_published: 2026-05-29
---

# Step 3.7 Flash

**Subtitle:** "The new frontier is agent efficiency."

**Tagline:** "Multimodal Understanding & Action｜Web & Visual Search Enhancement｜Reliable Tool Use & Orchestration｜Agent Ecosystem Compatibility"

196B total parameters, 11B active parameters, plus 1.8B ViT for vision. Multimodal.

## Key Features

1. **Native Multimodal Understanding & Acting** — Processes product UIs, documents, charts, and natural scenes, then generates code or invokes tools based on visual input.

2. **Web & Visual Search Enhancement** — Extended web search reach with deeper follow-up; visual search handles "long-tail entities, freshly emerged concepts."

3. **Reliable Tool Use & Orchestration** — Drives terminals, browsers, Office tools, and search "staying coherent however long the run gets." Claims "Less drift, fewer broken toolcalls, fewer failed runs."

4. **Agent Ecosystem Compatibility** — Works with "mainstream harnesses (Claude Code, KiloCode, Hermes Agent, OpenClaw) and Skills" with lower integration cost.

## Benchmark Scores

### Agentic Coding

**SWE-Bench Pro:**
- Step 3.7 Flash: 56.3 (196B params)
- Step 3.5 Flash: 51.3 (196B)
- DeepSeek V4 Flash: 55.6 (284B)
- Gemini 3.5 Flash: 55.1 (unknown params)
- GPT 5.5: 58.6
- Claude Opus 4.7: 64.3

**Terminal-Bench 2.1:**
- Step 3.7 Flash: 59.5 (196B)
- Step 3.5 Flash: 53.4 (196B)
- DeepSeek V4 Flash: 62.0 (284B)
- Gemini 3.5 Flash: 76.2
- GPT 5.5: 82.7
- Claude Opus 4.7: 69.4

### Step-SWE-Bench Results — Per-Harness

**Step 3.7 Flash:**
- Hermes Agent: 67.50%
- OpenClaw: 67.00%
- Claude Code: 71.50%
- KiloCode: 67.50%
- OpenCode: 64.50%
- RooCode: 64.50%
- Average: 67.08%

**Step 3.5 Flash:**
- Hermes Agent: 60.00%
- OpenClaw: 47.00%
- Claude Code: 73.00%
- KiloCode: 59.00%
- OpenCode: 57.00%
- RooCode: 43.00%
- Average: 56.50%

### Advisor Mode

With Advisor Mode (small executor escalates to frontier advisor only when needed), Step 3.7 Flash reaches "97% of Claude Opus 4.6's coding performance at roughly one-ninth the per-task cost" — "$0.19 v.s. $1.76 per task."

**SWE-Bench Verified Score vs. Cost:**
- Step 3.7 Flash + Advisor ($0.12): 73.7%
- Step 3.7 Flash + Advisor ($0.19): 76.3%
- Claude Opus 4.6 ($1.76): 78.7%

### Multimodal

**SimpleVQA (with Tool):**
- Step 3.7 Flash: 79.2
- GLM 5V Turbo: 78.2
- Kimi K2.6: 78.2
- GPT 5.5: 79.1

**V* (with Python):**
- Step 3.7 Flash: 95.3
- GLM 5V Turbo: 89.0
- Kimi K2.6: 96.9
- Gemini 3 Flash: 96.3

### General Agent

**GDPval:** Step 3.7 Flash: 45.8 (up from 28.0 on 3.5 Flash)
**Toolathlon:** Step 3.7 Flash: 49.5 (up from 33.3)
**ClawEval-1.1:** Step 3.7 Flash: 67.1 (up from 43.6)
**HLE (with Tool):** Step 3.7 Flash: 47.2 (up from 35.7)

### Full Benchmarks Table

| Benchmark | Step 3.7 Flash | Step 3.5 Flash | DeepSeek V4 Flash | Gemini 3.5 Flash | DeepSeek V4 Pro | GPT 5.5 | Claude Opus 4.7 | Kimi K2.6 | GLM 5.1 |
|---|---|---|---|---|---|---|---|---|---|
| Total Params | 196B + 1.8B (ViT) | 196B | 284B | — | 1.6T | — | — | 1T | 754B |
| Active Params | 11B | 11B | 13B | — | 49B | — | — | 32B | 40B |
| Multi-modal | ✓ | ✕ | ✕ | ✓ | ✕ | ✓ | ✓ | ✓ | ✕ |
| HLE w. tool | 47.2% | 35.7% | 45.1% | 40.2% | 48.2% | 52.2% | 54.7% | 54.0% | 52.3% |
| BrowseComp | 75.8% | 69.0% | 73.2% | — | 83.4% | 90.1% | 79.3% | 83.2% | 79.3% |
| deepsearchQA (F1) | 92.8% | 85.5%* | 90.6%* | — | — | 94.0%* | 91.7%* | 92.5% | 91.2%* |
| deepsearchQA (acc) | 81.7% | 73.4% | 79.8%* | — | — | 85.3%* | 82.3%* | 83.0% | 81.3%* |
| ResearchRubrics | 71.7% | 65.3% | 66.2%* | 63.6%* | 68.3%* | 61.5%* | 73.9%* | 63.0%* | 67.9%* |
| Toolathlon | 49.5% | 33.3% | 52.8%* | 56.5% | 56.6%* | 60.2%* | 65.4%* | 54.6%* | 48.1%* |
| Claweval-v1.1 | 67.1% | 43.6% | 57.8% | — | 59.8% | — | — | 62.3% | 62.3% |
| GDPval-Stirrup | 1415.8 | 1055.0 | 1414.0 | 1656.0 | 1554.0 | 1769.0 | 1753.0 | 1481.0 | 1535.0 |
| SWE-MTLG | 72.4% | 67.4% | 73.3% | — | 76.2% | — | 80.5% | 76.7% | — |
| SWE-Bench Pro | 56.3% | 51.3% | 55.6%* | 55.1% | 55.4% | 58.6% | 64.3% | 58.6% | 58.4% |
| SWE-Bench Verified | 76.5% | 74.4% | 79.0%* | — | 80.6% | — | 87.6% | 80.2% | — |
| Terminal-Bench 2.1 | 59.6% | 53.4% | 62.0%* | 76.2% | 72.0% | 82.7% | 69.4% | 66.7% | 69.0% |
| AA-LCR (avg@16/acc) | 63.9% | 45.5% | 63.7% | 71.0% | 66.3% | 74.3% | 70.3% | 69.1% | 64.9% |

### Visual Recognition Benchmarks

**Visual Recognition with Visual Search:**
| Benchmark | Step 3.7 Flash | Kimi K2.6 | GLM 5V Turbo | GPT 5.5 |
|---|---|---|---|---|
| SimpleVQA | 79.16% | 78.24%* | 78.20% | 79.11%* |
| WorldVQA | 58.10% | 55.98%* | 47.81%* | 54.58%* |
| BC-VL | 58.96% | 57.12%* | 51.90%* | 65.68%* |

**Visual Perception with Python Tool:**
| Benchmark | Step 3.7 Flash | Kimi K2.6 | GLM 5V Turbo | Gemini 3 Flash |
|---|---|---|---|---|
| V* | 95.29% | 96.90% | 89.00% | 96.30% |
| HR-Bench 4K | 89.13% | 91.25%* | 84.62% | 94.50% |
| HR-Bench 8K | 86.34% | 90.13%* | 83.12% | 94.80% |
| VisualProbe | 65.05% | 64.47%* | 53.01% | 69.90% |

### GUI Operation — Android Daily Benchmark

| Model | Score |
|---|---|
| Gemini 3 Flash | 63.21%* |
| Step 3.7 Flash | 61.87% |
| Kimi K2.6 | 53.36%* |
| GLM 5V Turbo | 51.68%* |

## Search Capabilities

Key scores: 47.20% on HLE with Tools (up from 35.68% text-only), 75.82% on BrowseComp, 92.82% F1 on DeepSearchQA, 71.68% on ResearchRubrics (ahead of GPT 5.5 at 61.50%).

## Emergent Behaviors

**Compositional generalization across visual and other tools** — The model "seamlessly combined visual tools with non-visual ones to accomplish complex tasks, despite never having been explicitly guided toward such compositional tool use during training."

**Code-and-GUI composition** — After writing frontend code, the model "autonomously turned to the GUI to test the page it had just produced" — inspecting, clicking, iterating. "This code-and-GUI compositional behavior was never explicitly demonstrated or rewarded during training, yet emerges robustly."

## Availability

- API: platform.stepfun.ai (global) and platform.stepfun.com (China)
- Web chat: stepfun.ai / stepfun.com
- Mobile: iOS and Android
- Third-party: OpenRouter, NVIDIA NIM
- Upcoming: DeepInfra, Fireworks AI, Modal

## Deployment

Cloud, data center, and local. Local/workstation requires "NVIDIA DGX Station, AMD Ryzen AI Max+ 395-based systems, and Mac Studio / Macbook Pro devices with at least 128GB unified memory."

## Ecosystem

vLLM, SGLang, Hugging Face Transformers, llama.cpp, NVIDIA Nemo ecosystem, NVIDIA NIM inference microservice.

## Links

- GitHub: github.com/stepfun-ai/Step-3.7-Flash
- HuggingFace: huggingface.co/stepfun-ai/Step-3.7-Flash
- ModelScope: modelscope.cn/models/stepfun-ai/Step-3.7-Flash

© 2026 StepFun. All rights reserved.
