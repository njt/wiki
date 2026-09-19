---
url: https://softwaredoug.com/blog/2025/01/21/llm-judge-decision-tree
title: "LLM Judge Decision Tree"
author: Doug Turnbull
date_fetched: 2026-09-19
date_published: 2025-01-21
topics:
  - guardrails-and-feedback-loops
  - local-and-open-source-inference
---

Doug Turnbull (search-relevance consultant; title here derived from the URL slug, as the fetched body carries no H1) asks whether a local LLM on his laptop can stand in for human relevance raters on the open WANDS e-commerce dataset from Wayfair. The goal is explicitly modest: not to replace human labels but "to at least be a reliable to flag what looks amiss / promising much faster without needing to always recruit humans" — and to do it without an OpenAI bill.

The method is deliberately decomposed. Instead of one big prompt judging two whole products, he runs many "dumb" pairwise prompts over single attributes — product name, taxonomic categorization, classification, description — in four variants: forced decision or allowed "Neither", and single-pass or double-checked (a decision only counts if swapping LHS/RHS gives the same answer). The result is a family of precision/coverage tradeoffs: on product names, forced single-pass decisions hit 75.08% precision at 100% coverage, while allowing "Neither" plus double-checking reaches 90.76% precision but only on 11.9% of pairs. "When the agent chickens-out (or isn't consistent in double checking) we improve precision, but lose recall." A single "uber prompt" with all four fields reaches 91.72% precision at 65.2% coverage (forced, double-checked).

The pivot of the piece: rather than voting-ensembling the per-attribute judgments, notice that the accumulated table — agent judgments as features, human preference as label — "is a Machine Learning problem!" He collects 7,000 examples by script, trains a plain scikit-learn decision tree to predict human preference from the LLM judgment variants, and applies a probability threshold (0.9) to buy precision back at the cost of recall. Reported spikes (explicitly caveated as un-cross-validated lab-notebook numbers): one feature set classifies every test label correctly but only covers 1.3% of pairs; another holds 96.5% precision across 40.5%. The exported tree is also an exploratory tool — it suggests, for example, that a search solution should weigh category before name.

The upshot is a compositional architecture: keep the LLM decisions "dumb, simple, and interpretable", then "combine their outputs into something smarter using traditional ML" — local LLMs as cheap, heavily cacheable ML feature generators, with "fast boring, old ML" finishing the job.
