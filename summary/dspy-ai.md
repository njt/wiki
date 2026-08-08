---
url: https://dspy.ai/
title: DSPy — Program, don't prompt, your LLMs
author: Stanford NLP / DSPy team
date_fetched: 2026-08-08
---

DSPy is a Python framework for building AI systems that replaces hand-written prompts with structured, typed signatures. The tagline — "Program, don't prompt, your LLMs" — captures the thesis: natural language prompts are a poor specification medium for probabilistic systems, and the alternative is to express tasks as typed inputs and outputs that the framework compiles into optimized prompts.

The framework has three core abstractions. **Signatures** define a task as typed inputs and outputs instead of prose prompts — portable, maintainable, and easy to iterate on. **Modules** control how a signature executes: reasoning strategies, ensembles, tool use, REPL interaction — without rewriting the task definition. **Optimizers** take examples and a scoring function, then automatically tune prompts until quality converges, eliminating the manual prompt-engineering grind that produces [[Prompt Debt]].

DSPy started at Stanford NLP in December 2022 and has grown into a research community. New optimizers and module types land here first before appearing in production systems.
