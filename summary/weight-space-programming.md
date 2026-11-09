---
url: https://rikiyat.github.io/blog/weight-space-programming/
title: "Sparks of Weight-Space Programming: LLMs can natively read & write neural network weights"
author: Rikiya Takehi, Jayden Teoh, Alex Zhang, Omar Khattab
date_fetched: 2026-11-09
date_published: 2026-11
topics:
  - ai-research-and-models
---

Takehi, Teoh, Zhang and Khattab (Berkeley Sky Lab / cmpnd orbit) report that frontier LLMs — "GPT-6 Astra" — can read and write neural network weights natively, in natural language, without training loops or gradient descent. They call this **Weight-Space Programming**: the LLM reasons about what parts of a network compute and edits the parameters directly.

The experiments escalate in scale and difficulty. Astra identifies which game four identically-shaped PPO policies were trained on, from raw weights alone. It reads middle-layer weights of 7.2M-parameter character LMs, names the training language and finds every induction (copying) head with no false positives, using a method from Elhage et al.'s circuit analysis — by multiplying weight matrices in its head, not running the model. It fixes a PPO car-ramp policy with a 3.5% weight edit that turns one failure into 1,024/1,024 successes, and writes MinAtar game-playing networks from scratch that beat PPO trained 100M steps (517 vs 15.8 in Breakout). With a Python-sandbox harness and a budget of forward passes, it writes a 1.5M-parameter Go program layer by layer and beats an equal-sized PPO net trained on 20M games.

On real models: Astra spots a reasoning-loop failure in Qwen3.5-0.8B (which the model card itself warns about), fixes it by editing 0.016% of the weights — MATH-500 rises 7.9%→48.1%, GSM8K 13.8%→67.1%, and IFEval triples to 62.3% since the loops weren't math-specific. A telling negative result: on Gemma 3 270M, which rarely loops, Astra could not improve math at all — "editing weights could remove an obstacle that hid an ability, but it could not add an ability the model lacked." Another partial failure: from weights alone it could not distinguish a sycophantic fine-tune from its twin, but given attention patterns it identified the sycophant in all pairs.

All logs are released (github.com/RikiyaT/lm_understand-logs), the edited Qwen is on Hugging Face, and the authors are building a benchmark and harness to turn "black magic into a rigorous, reproducible engineering discipline."
