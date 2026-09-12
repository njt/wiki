---
url: https://mistral.ai/news/shieldstral/
title: Shieldstral
author: Mistral AI
date_published: 2026
date_fetched: 2026-08-06
topics:
  - guardrails-and-feedback-loops
---

Shieldstral is Mistral AI's 3B-parameter open-weights multimodal safety classifier, released under Apache 2.0. It frames content moderation as a binary question-answering task: a policy is supplied as a plain-language query at inference time, and the model returns a calibrated yes/no safety score from a single forward pass. This policy-adaptive design means one checkpoint adapts to new deployment contexts without retraining — a departure from traditional guardrail models that bake fixed taxonomies of harm categories into their weights.

Shieldstral matches or outperforms open guard models up to 7× its size across text safety, refusal detection, policy adaptability, and multimodal benchmarks, while running on a single 16GB NVIDIA GPU. It unifies prompt classification, response moderation, refusal detection, and toxicity detection into a single interface that handles text, images, and text+image content.

The model was trained by unifying heterogeneous public safety datasets into a shared instruction–query–document format, constructing contrastive policy pairs to teach discrimination rather than memorization, supplementing scarce visual safety data with general-purpose image datasets as high-quality negatives, and merging complementary LoRA checkpoints via SLERP. Mistral built it end-to-end on Forge, their internal training platform.
