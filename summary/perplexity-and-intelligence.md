---
url: https://openai.com/index/perplexity-and-intelligence/
title: "Perplexity and Intelligence"
author: OpenAI (Johannes Heidecke et al., unsigned front-of-house post)
date_fetched: 2026-11-26
date_published: unknown (2026)
topics:
  - ai-research-and-models
  - guardrails-and-feedback-loops
---

OpenAI proposes cross-entropy loss (perplexity) as the industry's shared metric for tracking pretraining progress — and, more provocatively, as the basis for common capability limits that labs commit not to cross until monitoring capability catches up. To start, OpenAI publishes its own internal log-perplexity numbers on its main evaluation dataset, a disclosure the post admits hands competitors a picture of where it stands.

The argument runs: modern AI development is mostly *harnessing* the intelligence that emerges from training a compressor on a huge corpus; the pretraining loss is the "intelligence substrate", and it has predicted capability jumps remarkably well. GPT-3.5 at ~0.99 log-perplexity made fluent conversation possible; GPT-4 at ~0.91 enabled chain-of-thought reasoning; below 0.8 brought transformative cybersecurity and mathematics capability. The next transformative window is projected around 0.68–0.7.

The urgency comes from a claimed acceleration: an internal model at ~0.76 log-perplexity began producing "groundbreaking" mathematics results, and the post extrapolates that AI could avalanche fundamental pretraining research as soon as 2027. Hence the ask: shared transparency standards, joint red-teaming norms, and explicit perplexity-based limits tied to concrete monitoring milestones (e.g. a successor to CoT monitors). The post is candid about the metric's problems — cross-lab comparability, detachment from actionable tasks, blindness to chain-of-thought gains, and an unknown entropy floor — but argues perplexity has outlasted every benchmark that saturated around GPT-4.
