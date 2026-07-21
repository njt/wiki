---
url: https://magazine.sebastianraschka.com/p/controlling-reasoning-effort-in-llms
title: Controlling Reasoning Effort in LLMs
author: Sebastian Raschka
date_fetched: 2026-07-21
date_published: 2026-07-18
---

# Controlling Reasoning Effort in LLMs

Sebastian Raschka surveys how reasoning effort can be controlled in large language models, from the definition of reasoning models through training techniques (RLVR, SFT) to the specific implementations across six open-weight model families (DeepSeek V4, Nemotron 3 Ultra, Kimi K2.5, GLM-5, Qwen3, Inkling). Published in his "Ahead of AI" newsletter on Substack, July 18, 2026.

## 1. What Is a Reasoning Model?

A "reasoning model" means a model that outputs an intermediate reasoning trace — it works through a question step by step — not that it literally reasons like humans. The term describes output format, not cognitive process.

## 2. Training and Inference Scaling

Two ways to improve reasoning task performance: training scaling and inference scaling.

**RLVR (Reinforcement Learning with Verifiable Rewards):** Popularized by DeepSeek-R1, RLVR uses reward signals like `0=incorrect` and `1=correct` for domains with objectively verifiable answers (math, code). The reasoning trace itself was not used for training updates — only the final answer correctness mattered.

**"Aha" moments:** During RLVR training, models sometimes realize a mistake mid-trace and self-correct. These emergent behaviors appear without being explicitly trained for.

## 3. Think Tokens Are Cosmetic

The `<think>` and `</think>` tags are "not giving the model the ability to 'think' or reason or reason better." They are implemented via a format reward during RLVR training — the model learns to wrap its reasoning in tags because it's rewarded for doing so, not because the tags enable reasoning. The tags are a presentation layer, not a mechanism.

## 4. Reasoning Mode On/Off Switches

Early reasoning models like DeepSeek-R1 lacked a toggle — they were verbose even for simple prompts ("what is the capital of France?" would trigger a full reasoning trace).

**Qwen3** introduced hybrid approaches where `enable_thinking=True` or `enable_thinking=False` controls behavior through the tokenizer. "Thinking Mode Fusion" during training taught the model to switch modes based on the presence or absence of a thinking token.

## 5. How Reasoning Effort Settings Work

**GPT-5.6** exposes six effort settings (Light to Ultra). Evidence from open-source gpt-oss models shows effort is toggled via system prompts, not architectural changes.

Two primary training implementations:

1. **RLVR with length penalties per effort level:** Train a single policy with different length penalties corresponding to different effort levels. Higher effort = lower length penalty = longer reasoning traces.

2. **SFT after RLVR:** Train the base reasoning model with RLVR, then use supervised fine-tuning — pair prompts with target responses at specific effort levels — to teach the model to produce appropriate-length responses on command.

**Inkling's continuous effort:** Inkling uses continuous effort values (0.2–0.99) rather than discrete levels. The reward function is conceptually: R(e) = R_task - λ(e) × N_tokens, where the length penalty λ varies continuously with the requested effort level e.

## 6. Open-Weight Models: A Survey

| Model | Approach |
|-------|----------|
| **DeepSeek V4** | Separate effort specialists with different context windows |
| **Nemotron 3 Ultra** | Learned modes with hard budgets; random-budget truncation during training |
| **Kimi K2.5** | Toggle method alternating budgeted and unconstrained RL phases |
| **GLM-5** | Turn-level and interleaved thinking via SFT |
| **Qwen3** | Mode fusion and inference-time truncation |
| **Inkling** | Continuous effort conditioning in RL with variable length penalties |

## 7. Conclusion

Raschka identifies automatic effort selection as "the holy grail" but predicts reasoning effort "will remain an explicit model input" delivered through system prompts. Agent wrappers may increasingly infer appropriate settings automatically — making effort selection an orchestration concern rather than a user-facing control.
