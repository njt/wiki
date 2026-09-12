---
url: https://magazine.sebastianraschka.com/p/controlling-reasoning-effort-in-llms
title: "Controlling Reasoning Effort in LLMs"
author: Sebastian Raschka
date_fetched: 2026-07-21
date_published: 2026-07-18
topics:
  - ai-research-and-models
---

Raschka surveys how reasoning effort is controlled in LLMs, from definitions through training to concrete implementations across six open-weight model families. A "reasoning model" is defined by output format — it produces intermediate reasoning traces — not by any claim about human-like cognition.

Two training levers enable reasoning control. **RLVR** (reinforcement learning with verifiable rewards) trains on outcome correctness alone; the model discovers step-by-step traces emergently. The `<think>` tags are cosmetic — learned via format rewards, not a mechanism that enables reasoning. **SFT after RLVR** then teaches the model to produce traces of specific lengths on command by pairing prompts with effort-level-appropriate responses.

Effort control takes several forms across the surveyed models. GPT-5.6 uses system prompts with discrete levels (Light through Ultra). Qwen3 introduced a tokenizer-level on/off switch via "thinking mode fusion." Inkling uses continuous effort values (0.2–0.99) with a length penalty that varies continuously with the requested level. Other approaches include separate effort specialists (DeepSeek V4), hard token budgets (Nemotron 3 Ultra), and alternating budgeted/unconstrained RL phases (Kimi K2.5).

Raschka's conclusion: automatic effort selection is "the holy grail," but reasoning effort will likely remain an explicit model input delivered through system prompts, with agent wrappers increasingly inferring appropriate settings — making effort selection an orchestration concern rather than a user-facing control.
