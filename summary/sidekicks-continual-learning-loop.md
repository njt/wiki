---
title: "Sidekick's Continual Learning Loop"
url: https://shopify.engineering/sidekicks-continual-learning-loop
author: Shopify Engineering
date_fetched: 2026-08-14
date_published: unknown
section: "AI Research & Models"
topics:
  - ai-research-and-models
---

# Sidekick's Continual Learning Loop

Shopify Engineering's field report on the flywheel they run to turn production experience into better model weights. Frontier models are the fastest way to launch, but they're frozen, general-purpose, and too expensive to serve at Shopify's scale. The answer is a continual learning loop that compresses production traffic into a specialized model — faster, cheaper, and better than the frontier baseline.

## The Loop

1. **Define quality as a rubric** — completeness, execution, response quality, safety, with concrete score anchors. Two expert annotators blind-score 25 random samples; Cohen's kappa below ~0.2 means the rubric is ambiguous. Inter-annotator agreement is the ceiling for any judge.
2. **Calibrate a judge** with DSPy, using reflection-based optimizers (GEPA, Agentic Context Engineering) to turn the rubric into an offline metric that runs on infinite production data. Backtest against known A/B wins and run targeted degradation tests.
3. **Improve the frontier baseline with autoresearch** — an agent proposes edits to prompts, tool definitions, and harness code, evaluates against the judge, keeps or discards.
4. **Mine hard negatives** — low-scoring production conversations are repaired by a panel of frontier reasoning models (an arbiter merges critiques into a repair instruction), replayed, and re-scored. Passing replays become RL trajectories; failures go to Toloka's human annotators.
5. **Two-stage training** — supervised fine-tuning on healed trajectories (including reasoning, i.e. chain-of-thought distillation), then GRPO with the calibrated judge as reward. Runs daily on both new and prior data to limit drift and forgetting.
6. **Compress the prompt** — gist tokens reproduce a 6,000-token system prompt as ~1,500 learned tokens with no measured quality loss.

## Results (GraphQL agent, ~2,000 req/min)

- Specialized model surpasses frontier-model performance.
- Serving cost drops from an estimated $27M/year to ~$1M — a 96% reduction.
- Time-to-first-token down ~19%, end-to-end latency down ~38% at 350 req/min.
- ~14% fewer GPUs for the same traffic.

## Key Insight

Production knowledge accumulates in discrete artifacts (prompts, code, routing rules) while the model's weights stay frozen. The flywheel's durable advantage is the loop itself: each cycle begins with a more capable *model*, not merely a more elaborate harness.
