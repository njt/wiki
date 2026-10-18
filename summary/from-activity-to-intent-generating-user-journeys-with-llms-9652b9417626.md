---
url: https://medium.com/pinterest-engineering/from-activity-to-intent-generating-user-journeys-with-llms-9652b9417626
title: "From Activity to Intent: Generating User Journeys with LLMs"
author: Lin Zhu, Manan Kalra, Byung Eun Jeon, Ye Liu, Xiangyi Chen, Jaewon Yang, Jinwen Xu, Tingting Zhu, Sudarshan Lamkhede (Pinterest Engineering)
date_fetched: 2026-10-18
date_published: 2026-10-10
topics:
  - ai-research-and-models
  - guardrails-and-feedback-loops
---

Pinterest replaced a multi-stage keyword-clustering pipeline for inferring user "journeys" (long-horizon goals like planning a kitchen remodel) with a single generative model: a fine-tuned 4B Qwen3 that reads a chronological activity log and emits strict JSON — a ranked list of journey names in the user's locale. The legacy pipeline had real lift (email click +88%, push open +32%) but compounded stage-wise errors: fragmented coherent goals into separate journeys, fell back to generic buckets ("Art", "Recetas"), and let noise occupy top slots. Generation collapses all of that into one model, but "the prompt becomes the ranking policy and the safety policy" — so prompt changes are treated as launches.

Frontier models make good teachers but can't be served to hundreds of millions of users, so Pinterest distilled them into Qwen3 students (0.6B–8B); the 4B model hit the quality/throughput sweet spot. Quality plateaued after a few thousand examples — diverse, hard cases mattered more than volume. Serving is NVIDIA Dynamo in front of vLLM on L40S GPUs with prefix caching and KV-aware routing: ~100 GPUs sustain ~775 req/s, median 1.2s. The cheapest throughput lever wasn't a GPU — it's skipping users inferred within the last couple of days.

Evaluation runs at three levels because there is no ground-truth label for a latent goal: side-by-side same-user comparison (including honest misses), LLM-scored diagnosis on axes like "support from the log" (a fluent name not grounded in the log is a failure), and online experiments (+1.1% email CTR, +1.3% push opens, sitewide engagement and retention gains). Work in progress includes semantic IDs (RQ-VAE-compressed Pin codes replacing tags, at the cost of human-readability), offsite conversion signals, one model emitting multiple products (multi-granularity journeys, notification copy), and preference optimization from explicit Yes/No feedback formulated as "feedback-as-edit."
