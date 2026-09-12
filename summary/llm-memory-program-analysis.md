---
url: https://pwning.systems/posts/llm-memory-program-analysis/
title: "I accidentally turned LLM memory into program analysis"
author: unknown
site: pwning.systems
date_fetched: 2026-09-04
date_published: unknown
topics:
  - agent-memory-and-context
---

# I accidentally turned LLM memory into program analysis (precis)

A vulnerability researcher trying to keep LLM agents from losing track of long investigations ends up building **Lemmalog**, a Datalog engine for agent memory. The core reframe: LLM "memory" is really two different problems — *retrieval* ("what past information is relevant?") and *state maintenance* ("given everything learned, what is currently true?"). Vector databases are good at the first; Lemmalog experiments with solving the second.

The motivating failure mode is a multi-hour vulnerability investigation where the model resurrects a disproven hypothesis or keeps reasoning from an observation that was later shown false. Telling an LLM something is wrong does not make it stop believing the conclusions that depended on it. The author's realization is that this is exactly the problem program analysis and deductive databases already solve: facts plus rules derive a fixed point, and when an input fact changes you incrementally invalidate only the affected results — with provenance explaining *why* each conclusion holds.

Lemmalog therefore splits the work. The LLM stays responsible for the fuzzy part — converting source code, debugger output, and natural-language notes into structured facts. The deterministic engine handles the rest: adding facts, applying rules, deriving conclusions, **retracting** conclusions when their supporting facts disappear, tracking **provenance** (so an agent can be asked *why* it believes something), and associating facts with **validity intervals** so "we used to believe X" and "X is false now" can coexist without contradiction.

Benchmarked on LongMemEval and LoCoMo with standard readers and judges: Lemmalog hits 0.463 F1 on LongMemEval and 0.533 on LoCoMo — roughly competitive with, but not beating, PropMem — while passing the reader ~38× less context than full-transcript prompting on LongMemEval. It tops the field on LongMemEval's Knowledge Update category (0.579), the situation closest to maintained program state. The author is explicit this proves the idea is "not completely stupid," not that Datalog has solved LLM memory; inference remains weak (0.164 on LoCoMo), because flattening conditional preferences into unconditional tuples throws away nuance before the engine ever sees it.

The most interesting result is not the number but how it improved: from 0.226 to 0.463 on LongMemEval by fixing entity identity, date representation, semantic aliases, a plural stemmer that failed to match "owns" to "own," and a reader accidentally taught to refuse synthesis. None of it required a bigger model — only better-maintained state around one.
