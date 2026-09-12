---
url: https://www.cmpnd.ai/blog/let-the-model-write-the-code.html
title: Let the Model Write the Code
author: Michael Isaac (guest post, implemented Flex for DSPy)
site: cmpnd.ai
date_fetched: 2026-08-08
topics:
  - agent-architecture
---

DSPy introduces `Flex`, a module that exposes not just a program's instructions but its *code* to the optimizer. Where previous DSPy optimizers like BootstrapFewShot and MIPROv2 could only rewrite prompts and pick examples, Flex lets GEPA — a reflective optimizer — rewrite the module's Python source, authoring helper functions, routing logic, and decomposed signatures alongside the prompt.

The central claim, backed by a worked geospatial conflation benchmark: giving the optimizer a second lever (code) produces programs that are simultaneously more accurate, cheaper, and faster. At λ=0 (no penalty on LLM calls), Flex + GEPA reached 95.0% accuracy vs. 90.4% baseline while routing 75% of records through deterministic Python — 28% cheaper and 40% faster. At higher λ penalties, accuracy held at parity with the baseline at a fraction of the cost (λ=0.4: 92.1% accuracy, $0.01 per thousand records, one model call across 240 records).

The generated code follows a consistent three-stage pattern: normalize (strip noise), compare (bin into confident/unsure buckets), decide (rules per bucket, with the LLM as last-resort fallback). Four moves recur across tasks: decomposition, method selection (code vs. model per step), routing (easy cases down the cheap path), and evolution (refining the internals once the structure settles).

Flex executes generated code inside a sandboxed interpreter by default — it never runs in your process. Only predictor calls and explicitly provided tools bridge to the host. The thesis: models are now good enough at programming that we can continually compile our harnesses as new data, models, and tactics arrive.
