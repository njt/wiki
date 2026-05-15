---
title: "siftrank"
url: https://github.com/noperator/siftrank
date_fetched: 2026-05-14
section: "LLMs"
---

# SiftRank: LLM-Based Document Ranking

Go-based tool implementing the SiftRank algorithm for ranking datasets by relevance to user-defined prompts.

## Core Algorithm

1. Stochastic sampling: Randomly divides datasets into manageable batches
2. Inflection detection: Identifies natural breakpoints separating highly relevant items
3. Fixed complexity: Caps LLM calls to maintain linear computational scaling
4. Iterative refinement: Repeats comparisons until scores stabilize

## Key Features

- No fine-tuning required -- works with off-the-shelf models like GPT-4o
- Deterministic results across runs
- Cost-effective: typically seconds for modest datasets, costing pennies
- JSON and text support with template customization
- Real-time terminal visualization of ranking progress
- Profile-based configuration with environment variable support

## Technical Capabilities

- Batch processing (default 10 items)
- Token-aware processing (default 128K tokens per batch)
- Concurrent API calls (default 50)
- Convergence detection with early stopping
- Elbow method for score analysis (curvature or perpendicular)
- Support for multiple OpenAI models and compatible APIs

## Example

Successfully ranked 100 sentences by relevance to "time" in 7 seconds, correctly identifying time-related items as top results.

Written entirely in Go with no external dependencies beyond OpenAI API calls.
