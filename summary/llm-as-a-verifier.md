---
url: https://arxiv.org/pdf/2607.05391
title: "LLM-as-a-Verifier: A General-Purpose Verification Framework"
author: Jacky Kwok, Shulu Li, Pranav Atreya, Yuejiang Liu, Yixing Jiang, Chelsea Finn, Marco Pavone, Ion Stoica, Azalia Mirhoseini
date_fetched: 2026-07-18
date_published: 2026-07
topics:
  - ai-research-and-models
---

A training-free, general-purpose verification framework that produces fine-grained
continuous scores for agentic tasks. The core insight is that standard LM judges
collapse scoring distributions into coarse discrete scores, producing high tie
rates (27% on Terminal-Bench V2) and poor discrimination between good and bad
trajectories. The paper shows that models often *can* solve tasks — they just
can't reliably tell which of their attempts succeeded.

The framework scales verification along three complementary dimensions. **Score
granularity** expands the decoder's scoring token set to produce continuous
scores via logit expectation rather than taking the argmax discrete token.
**Repeated evaluation** averages K independent evaluations to reduce variance.
**Criteria decomposition** replaces a single monolithic rubric with an ensemble
of sub-criteria (e.g., for code: Specification, Output, and Errors separately).

To rank candidate solutions efficiently, the authors introduce **Probabilistic
Pivot Tournament (PPT)** — a ring-pass + pivot-round algorithm that reduces
pairwise comparison budget from O(N²) to O(Nk) while approaching full
round-robin accuracy.

The method achieves state-of-the-art results across four benchmarks: Terminal-Bench
V2 (86.5%), SWE-Bench Verified (78.2%), RoboRewardBench (87.4%), and MedAgentBench
(73.3%). Beyond ranking, the continuous verifier score serves as a task progress
estimator (correlating strongly with chronological step index) and as a dense
reward signal for RL, improving sample efficiency ~1.8× in SAC and ~1.1× in GRPO.

The framework is plug-and-play: applied identically across coding, robotics, and
medical domains with no per-domain fine-tuning. A two-stage pipeline recovers
most of the benefit for closed models that don't expose logprobs, and TurboAgent
provides a drop-in extension for Claude Code and OpenAI-compatible clients.
