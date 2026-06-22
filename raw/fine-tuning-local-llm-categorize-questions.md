---
url: "https://www.teachmecoolstuff.com/viewarticle/fine-tuning-a-local-llm-to-categorize-questions"
title: "Fine Tuning a Local LLM to Categorize Questions"
author: "Torgeir Helgevold"
date_published: "2026-06-16"
date_fetched: "2026-06-22"
---

# Fine Tuning a Local LLM to Categorize Questions

Torgeir Helgevold, 16 Jun 2026. teachmecoolstuff.com.

## Overview

The author describes a personal project building a household Q&A chatbot that uses RAG with metadata-aware vector search. Questions are pre-processed through a categorization step, tagging them with labels like "pool," "car," "hvac," or "cooking" to narrow vector search results to matching indexed entries.

## Models Used

Two local models: **Qwen 3:4B** for general answering, and **Qwen 3:0.6B** — a tiny 600M-parameter model — for classification. The central experiment tests whether this tiny model "can be finetuned into a reliable classifier of household questions."

## Finetuning Approach

Uses **Unsloth** with QLoRA. Initial dataset: ~850 entries split 70/15/15 into training, eval, and test sets.

Sample training data format:
```json
{ "question": "Who cleans our gutters at the house?", "category": "gutters" }
```

The author notes that "the default fine tuning parameters provided by Unsloth provide a very good starting point" and emphasizes that a good dataset matters more than tweaking hyperparameters, at least initially.

## Baseline Results (No Finetuning)

Battery of 131 integration tests:
- **13 correct out of 131 (~10% accuracy)**

Common failures: model overused broad labels like "electric" or "appliances," and invented categories not in the allowed list (e.g., "apartments").

## Finetuning — 1st Attempt

Same prompt format, finetuned model:
- **104 correct out of 131 (~79% accuracy)**

Two issues persisted: the model emitted fragments of correct category names (e.g., "ac" or "air" instead of "hvac"), and struggled with semantically overlapping categories — "water-based confusion from fountain, water heater and pool."

## Finetuning — 2nd Attempt

Rather than adding post-processing, the author made "a minor change to the prompt" — mapping categories to two-character opaque IDs with no semantic overlap. For example, `KK = hvac`, `OO = pool`, `QQ = water heater`.

The prompt instructs: "Return only the short label code from the list. Never return the category name, a number, a synonym, an explanation, or any other text."

- **120 correct out of 131 (~92% accuracy)**

Remaining failures: water heater questions misclassified as "pool" — still likely due to "the overlapping 'watery' meaning between those two categories." Plans to revisit training data.

Despite these misses, "the finetuned llm serves as a usable predictor" in the chatbot. A screenshot shows chat bubbles with automatically assigned category tags.

## GitHub Repositories

- [Fine-tuning script](https://github.com/thelgevold/fine-tuned-classifier/blob/main/fine-tuning/train_categories.py)
- [Main project repo](https://github.com/thelgevold/fine-tuned-classifier)
