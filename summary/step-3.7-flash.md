---
url: https://static.stepfun.com/blog/step-3.7-flash/
title: Step 3.7 Flash
author: StepFun
date_fetched: 2026-05-31
date_published: 2026-05-29
topics:
  - ai-research-and-models
---

# Step 3.7 Flash — See. Think. Act.

A high-efficiency Flash model for real-world agents. Up to 400 TPS. 196B MoE (11B active) + 1.8B ViT.

## Key Features

**Native Multimodal Understanding & Acting** — Understands images across the full range (product UIs, documents, charts, natural scenes) then writes code or calls tools to act on what it sees.

**Web & Visual Search Enhancement** — Web search reaches further with more sources and deeper follow-up. Visual search recognizes long-tail entities and freshly emerged concepts that other systems miss.

**Reliable Tool Use & Orchestration** — Drives terminals, browsers, Office tools, search, and beyond — staying coherent however long the run gets. "Less drift, fewer broken toolcalls, fewer failed runs."

**Agent Ecosystem Compatibility** — Works with Claude Code, KiloCode, Hermes Agent, OpenClaw, OpenCode, RooCode and Skills. Lower integration cost, less workflow rewiring.

## Benchmarks

| Benchmark | Step 3.7 Flash | Step 3.5 Flash | DeepSeek V4 Flash | Gemini 3.5 Flash | GPT 5.5 | Claude Opus 4.7 |
|-----------|---------------|---------------|-------------------|------------------|---------|-----------------|
| SWE-Bench Pro | 56.3 | 51.3 | 55.6 | 55.1 | 58.6 | 64.3 |
| Terminal-Bench 2.1 | 59.5 | 53.4 | 62.0 | 76.2 | 82.7 | 69.4 |
| SimpleVQA (with Tool) | 79.2 | — | — | — | 79.1 | — |
| V* (with Python) | 95.3 | — | — | 96.3 (3 Flash) | — | — |
| GDPval | 45.8 | 28.0 | 44.0 | 57.8 | 63.0 | 63.0 |
| Toolathlon | 49.5 | 33.3 | 52.8 | 56.5 | 60.2 | 65.4 |
| ClawEval-1.1 | 67.1 | 43.6 | 57.8 | — | 60.3 (5.4) | 70.8 (4.6) |
| HLE (with Tool) | 47.2 | 35.7 | 45.1 | 40.2 | 52.2 | 54.7 |

Note: On non-multimodal tasks, left panel compares Step 3.7 Flash with DeepSeek V4 Flash (open-source Flash-size scale), right panel places it alongside frontier closed-source models.

## Agentic Coding — Per-Harness Results

| Harness | Step 3.7 Flash | Step 3.5 Flash |
|---------|---------------|---------------|
| Hermes Agent | 67.50% | 60.00% |
| OpenClaw | 67.00% | 47.00% |
| Claude Code | 71.50% | 73.00% |
| KiloCode | 67.50% | 59.00% |
| OpenCode | 64.50% | 57.00% |
| RooCode | 64.50% | 43.00% |

Per-harness gap narrowed substantially vs Step 3.5 Flash (which had a 30-point spread: 73% Claude Code vs 43% RooCode).

## Advisor Mode

Step 3.7 Flash drives the trajectory end-to-end — calling tools, reading results, iterating — and consults a larger advisor model only at inflection points (planning, recovering from repeated failures). This is Step's implementation of the advisor strategy described by Anthropic.

**Score vs. Cost on SWE-Bench Verified:**
- Step 3.7 Flash (no advisor): 73.7% at $0.12/task
- Step 3.7 Flash + Advisor: 76.3% at $0.19/task
- Claude Opus 4.6: 78.7% at $1.76/task

97% of Opus 4.6 coding performance at ~1/9th the per-task cost.

## Enterprise Tasks

GDPval: 45.8% across 44 occupations. Toolathlon: 49.5% for multi-tool coordination. ClawEval-1.1: 67.1% for daily autonomous task execution. Passes at >98% across reasoning difficulty tiers on Tau2-bench Telecom.

Includes embedded case studies (manufacturing production scheduling, engineering heat treatment analysis) demonstrating long-horizon task execution with Excel generation, document parsing, and multi-tool orchestration.

## Gallery

8 examples: Landing Page, Heritage Building, Compositional Tool-use, Travel Guide, Deep Search, Draft to Code, Video to Summary, Sketch to Web Page.

**Emergent compositional tool use:** The model autonomously tests its own web output by opening a GUI browser, inspecting, clicking, and iterating — behavior never explicitly demonstrated or rewarded during training.
