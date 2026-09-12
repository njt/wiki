---
url: https://www.oreilly.com/radar/the-problem-is-prompt-debt/
title: "The Problem Is Prompt Debt"
author: Drew Breunig
date_published: 2025
date_fetched: 2026-08-07
site: O'Reilly Radar (originally published on Drew Breunig's blog)
topics:
  - guardrails-and-feedback-loops
  - specifications-as-the-product
---

Drew Breunig diagnoses a specific form of technical debt unique to AI systems: **prompt debt**. Natural language prompts enable rapid prototyping but become a trap as systems grow. The imprecision of prose paired with probabilistic models means every hot-fix added to a prompt risks regressing earlier instructions, makes the prompt illegible to teammates, and locks the application to a single model — because fixes tuned for one model's weights fail on others.

The root cause is structural: natural language was never meant as a specification language for engineering. Fighting a model's training weights (repeating instructions, all-caps threats) is a symptom of using the wrong tool for the job. Breunig documents this pattern across production systems — ChatGPT's image prompt instructs the model eight times not to reply; Claude Code tells Opus seven times to return multiple tool calls; Fable's system prompt restates one copyright rule six times.

The prevention strategy has two principles, both drawn from coding-agent best practices that evolved at the frontier: **(1) specify behavior with measurements, not prose** — evals, metrics, and typed specs are the hard edges that constrain probabilistic output; **(2) stop writing prompts by hand** — search the prompt space algorithmically using systems like DSPy and GEPA, holding prompts accountable to your measurements. Once prompts are generated and behavior is defined by metrics, model portability becomes achievable: evaluating a new model takes hours, not weeks.

Breunig frames this as the natural maturation of an engineering discipline: assembly gave way to compilers, hand-tuned queries gave way to planners, manual memory management gave way to garbage collectors. Prompt-writing is no different — coaxing the model with exactly the right words is a real skill, but to build reliable, improvable, portable systems we should not be hand-tuning prompts.
