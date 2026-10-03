---
url: https://allenai.org/blog/astabrief
title: "AstaBrief: an open model for generating scientific reports"
author: Allen Institute for AI (Ai2)
date_fetched: 2026-10-03
date_published: 2026
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

Ai2 built AstaBrief 8B, an open-weights model (fine-tuned from Qwen3-8B) that turns a research question plus retrieved literature excerpts into a fully cited scientific report. It now powers the "Fast mode" of Asta's report generation feature alongside the Claude-powered "Thinking mode", and the model and training data are open-sourced. The headline numbers: 51.1 seconds per report vs 178.5 for the Claude pipeline (~3.5× faster), with report quality competitive on citation and answer metrics.

The interesting engineering is in the data, not the architecture. They skipped RL (DR Tulu-style) in favour of plain SFT + DPO, betting that cheaper, debuggable training with excellent data beats expensive optimization with noisy data. SFT data came from 90K filtered real user queries turned into target reports by a mix of proprietary models (Claude 3.5/3.7, o3/o4-mini, GPT-4.1); DPO pairs were built by pitting the ScholarQA pipeline against alternative models, judged by GPT-4.1 and DeepSeek-R1 with both judges required to agree (95% human alignment).

The sharpest lesson: a simple citation-density filter — does the synthetic report consistently cite its claims? — beat more elaborate filter combinations. And for speed, they trained the model to emit the full report in one pass, bypassing the expensive snippet-summarisation/clustering stages of the Claude pipeline, with no quality loss.

They're also candid about the evaluation's limits: metrics measured relevance, coverage, and citation precision/recall, but not whether the model preserves the *scope* of claims (sample → population, past-tense finding → universal truth, description → recommendation). That richer evaluation of evidentiary faithfulness is named as future work.
