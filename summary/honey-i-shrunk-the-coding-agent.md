---
url: https://itayinbarr.substack.com/p/honey-i-shrunk-the-coding-agent
title: "Honey, I Shrunk the Coding Agent"
author: Itay Inbar
date_fetched: 2026-05-15
date_published: 2026-04-19
topics:
  - agent-architecture
  - local-and-open-source-inference
---

Coding Agent Adaptation Lets a 9B LLM Outperform 10x Larger Models on Aider Polyglot Benchmark

## Introduction

The author describes how coding agents are "built around assumptions of high autonomy, strong long-horizon planning, and reliable tool use" – assumptions suited to frontier models but not to small local models. He argues that when a small model fails inside a standard scaffold, those failures "may in fact be evidence of misalignment between the model and the agent wrapped around it."

The standard evaluation framing treats the scaffold as fixed and the model as variable, but the author inverts this: he holds the model constant (Qwen3.5-9B in Q4_K_M quantization) and varies the scaffold, comparing Aider's defaults against a custom scaffold called little-coder "whose design has been adapted specifically to the behavioral profile of a small local model."

A secondary finding: "even the Aider baseline reported here…scores above several larger models already on the public board," suggesting sub-10B models were "excluded from the coding-agent evaluation conversation prematurely."

## Methods

**Model setup:** Both systems use ollama/qwen3.5 at 9.7B parameters, Q4_K_M quantization (~6.6 GB weights), served via Ollama 0.20.5 in Docker on an RTX 5070 Laptop and 14900HX host. Context window: 32,768 tokens. Temperature: "0.3 for little-coder and 0 for Aider." No fine-tuning or LoRA applied.

**Benchmark:** "The Aider Polyglot suite in its full 225-exercise form, covering Python, Go, Rust, JavaScript, C++, and Java." Each exercise gets up to two attempts; a test-file audit confirmed only six exercises involved test edits, and the two that passed "were manually confirmed to be benign edits such as comment removal."

**Baseline setup:** Aider 0.86.2 run through its official benchmark harness. Defaults preserved for scaffold behavior. Two inference changes: context raised to 32,768 tokens; litellm timeout raised from 600s to 1,800s. A 3,600s wall-clock cap was added. Infrastructure fixes for path resolution and wall-clock capping were applied to run outside Docker, but "none of these changes touches Aider's prompts, coder logic, retry mechanics, or edit-parsing code."

**Agent loop:** little-coder inherits core structure from the SafeRL-Lab/clawspring agent substrate. Key adaptations:
- Write-tool guard refuses to operate on existing files, redirecting to Edit instead
- Thinking-budget cap at 2,048 tokens; exceeding it preserves partial reasoning and retries without thinking
- Workspace discovery via conditional-injection knowledge entry for coding keywords
- Reference material through tool skill cards (80–150 tokens each) and algorithm cheat sheets, capped at ~500 tokens per turn
- Malformed-output parser and quality monitor repairing tool calls and catching empty responses

The author summarizes: "little-coder turns each of them into infrastructure."

## Results

**Aider baseline:** Solved 43 exercises (19.11%) in 15h39m wall time. Language range: "12.8% on Go to 26.9% on C++." Described as "the first result I am aware of that places a sub-10B quantized local model at measurable performance on Aider Polyglot."

**little-coder:** Across two runs, "a mean of 45.56% (SD=0.94)." Four of six language tracks reproduced pass counts exactly; "79.1% produced the same pass-or-fail outcome in both runs."

**Scaffold mechanisms:** The Write guard "fires on approximately 57% of exercises." The thinking budget fires "approximately 0.90 times per exercise." Glob is "invoked on 67% of exercises and Read on 100%." The second-attempt retry path contributes "roughly 18% of all passes."

**Matched-model comparison:** little-coder solves 101, Aider solves 43. "69 are solved by little-coder and failed by Aider, and 11 are solved by Aider and failed by little-coder." The author notes: "The asymmetry is the scaffold signal."

**Language-level performance:** Strongest in Python (~53%) and Java (~52%), weakest in "Go at 38.5% and Rust at 30.0%." The widest scaffold gap was "in Python, at 35.3 percentage points, and narrowest in Rust, at 13.3 percentage points."

**First-attempt/retry:** "About 82% of little-coder's passes arrive on the first attempt, and the remaining 18% come from the second-attempt retry path." Retry value varies by language: in C++ it's worth 13.5%, in Java only 5.3%.

**Time economics:** Aider averages "141 seconds on passing exercises and 224 seconds on failing ones" (ratio ~1.6). little-coder averages "177 seconds on passing exercises and 492 seconds on failing ones" (ratio ~2.8). Pass-per-hour throughput favors little-coder at "approximately 4.7 passes per hour...compared to 2.75 for Aider."

## Discussion

The author states the core finding: "redesigning the scaffold around the behavioral profile of a small local model moves the pass rate from 19.11% to 45.56%."

He notes limitations: "transfer of these findings to SWE-bench, to real pull-request workflows, or to other small-model backbones has not been established." The observables "are not ablations" and formal per-mechanism ablations "are the natural next step."

Infrastructure challenges are acknowledged: "Ollama's runner subprocess was observed to die and be replaced mid-benchmark in contaminated runs."

Residual failures fall into patterns: "search failures that end in max-turn exhaustion," language regimes where "compile-time burden consumes the turn budget," and "algorithmic ceiling tasks" including "book-store, react, and zebra-puzzle" that both scaffolds consistently fail.

The concluding argument: "benchmark outcomes for coding agents are properties of model weights in interaction with the agent that surrounds them, rather than properties of model weights alone."

## GitHub Repository

https://github.com/itayinbarr/little-coder/tree/main
