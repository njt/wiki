---
url: https://danluu.com/ai-coding/
title: "Agentic test processes, LLM benchmarks, and other notes on agentic coding from Galapagos Island"
author: Dan Luu
date_fetched: 2026-07-29
date_published: 2026-07
topics:
  - guardrails-and-feedback-loops
---

Dan Luu reflects on how the testing practices from his decade at Centaur (a hardware company) apply to modern AI-assisted software development, and reports benchmark results on LLM coding performance including "caveman mode" and model comparisons.

The core argument: methodology around AI models matters far more than which model you use. At Centaur, dedicated test engineers (career parity with developers), no code review by default, randomized/property-based testing instead of hand-written unit tests, and a massive regression suite (~3 months wall-clock) produced fewer than one significant user-visible bug per year. Luu has since applied this methodology across every kind of software project thrown at it and found it works universally.

On LLMs and testing: LLMs are bad at writing tests when left to their own devices, but directing them to generate fuzzers turns up real, often serious bugs within minutes on most projects. Fuzzing beats LLMs for bug-finding latency, volume, and false positive rate. Reducing false positives from LLM-generated tests benefits from independent agents checking reproductions, contrarian review personas, artifact generation, and getting genuinely independent perspectives — nearly everything Luu tried to cut false positives worked.

On benchmarks: single-number summary benchmarks are "basically meaningless." Task variance within and across models is so large that changing a few tasks out of ~100 can flip which model appears to lead. Every contradictory claim about GPT-5.5 at release found support somewhere in Luu's three benchmarks. He notes Anthropic's business grew faster than OpenAI's during this period despite GPT-5.5 beating Opus on most benchmarks — the benchmarks aren't major determinants of user choice.

On "caveman mode" (a prompt style claimed to reduce tokens and speed up responses): after testing across multiple benchmarks, models, and effort levels, the overall difference averages out to be small enough that it isn't worth using.
