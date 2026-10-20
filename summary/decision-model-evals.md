---
url: https://hamel.dev/blog/posts/decision-model-evals/
title: "How To Use Decision Models for Evals"
author: Hamel Husain
date_fetched: 2026-10-20
date_published: 2026-10-17
topics:
  - guardrails-and-feedback-loops
  - ai-research-and-models
---

Husain benchmarks six decision models (TypeSafe's Jev, Perplexity's Decider, Liquid's d1 and d1-ultra, Cloudflare's Clef Flash, and OpenAI's Decisions API) as cheap alternatives to LLM judges for automated evals, using 98 production traces from an apartment-leasing agent labelled for failures.

The core argument is classical ML discipline applied to a new model class: never use the default 0.50 decision threshold, tune it against labelled data with explicit minimum precision/recall requirements, and pick the threshold to match your business tradeoff (criminal-trial precision vs cancer-screening recall) rather than blindly maximising F1. On cost-vs-performance, OpenAI Decisions (GPT-6 Luna) won outright (F1 0.87 at 3.72¢/100 runs, ~4.5× Perplexity's cost at F1 0.79). On calibration, several models' raw probabilities were badly off (Clef Flash predicting 8% where 60% of traces actually failed), and Platt scaling fixed most but not all.

The practical takeaway: model performance varies substantially across tasks and datasets, so measure on your own data; narrow the judge's scope to one failure mode per judge; and the copy-paste prompts for threshold tuning and calibration (given to a coding agent, reusing saved scores with no new model calls) are the reusable artifacts. Husain closes by noting none of this is new — it's standard classifier evaluation — which is exactly the point for ML engineers wondering where they fit in the AI age.
