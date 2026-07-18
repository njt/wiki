---
url: https://arxiv.org/pdf/2607.05391
title: "LLM-as-a-Verifier: A General-Purpose Verification Framework"
author: Jacky Kwok, Shulu Li, Pranav Atreya, Yuejiang Liu, Yixing Jiang, Chelsea Finn, Marco Pavone, Ion Stoica, Azalia Mirhoseini
date_fetched: 2026-07-18
date_published: 2026-07
---

# LLM-as-a-Verifier: A General-Purpose Verification Framework

Jacky Kwok, Shulu Li, Pranav Atreya, Yuejiang Liu, Yixing Jiang, Chelsea Finn, Marco Pavone, Ion Stoica, Azalia Mirhoseini

Stanford University, UC Berkeley, NVIDIA Research

arXiv: 2607.05391v2

License: CC BY 4.0

## Abstract

Verification is a new scaling axis for improving LLMs. This paper presents LLM-as-a-Verifier, a general-purpose verification framework that provides fine-grained feedback for agentic tasks without requiring additional training. Unlike standard LM judges that produce discrete scores, it computes the expectation over the distribution of scoring token logits for continuous scores. The approach scales along three dimensions: score granularity (better separation between positive and negative solutions), repeated evaluation (variance reduction), and criteria decomposition (complexity reduction). They introduce a cost-efficient ranking algorithm, Probabilistic Pivot Tournament (PPT), for selecting the best candidate solution using continuous scores.

## Benchmarks (all SOTA)

- Terminal-Bench V2: 86.5%
- SWE-Bench Verified: 78.2%
- RoboRewardBench: 87.4%
- MedAgentBench: 73.3%

## Additional Contributions

- Fine-grained signals serve as a proxy for estimating task progress (VOC metric)
- TurboAgent: drop-in extension for Claude Code and OpenAI-API compatible clients
- Dense reward for RL: improves sample efficiency of SAC (~1.8×) and GRPO (~1.1×)
- Two-stage pipeline for closed models that don't expose logprobs

## Code

- GitHub: github.com/llm-as-a-verifier/llm-as-a-verifier
- Website: llm-as-a-verifier.com

## Full Paper Content

### 1. Introduction

Standard LM judges collapse scoring distributions into coarse discrete scores, producing ties and poor discrimination. Learned reward models are constrained by training data and fail to generalize across domains. When repeatedly sampled with an oracle verifier, models achieve 98.9% on Terminal-Bench V2 — meaning most models already *can* solve tasks, but lack a verifier reliable enough to select correct trajectories from incorrect ones. Standard judges produce a 27% tie rate on this benchmark.

### 2. Methodology

#### Core Innovation: Logit Expectation

Instead of taking the highest-probability discrete score token, the framework computes the expectation over the distribution of scoring token logits to produce continuous scores:

R(x,τ) = (1/CK) Σ_c Σ_k Σ_g p_θ(v_g|x,c,τ) · φ(v_g)

Where C = number of evaluation criteria, K = repeated evaluations, G = score token granularity.

These continuous rewards are converted into pairwise preferences via a Bradley-Terry model:

P(τ_i ≻ τ_j | x) = 1 / (1 + exp(-(R(x,τ_i) - R(x,τ_j))))

#### The Three Scaling Dimensions

**1. Score Granularity (G):** Enlarging the ordered token set from 1 to 20 gives the decoder a finer space to project internal beliefs. SNR increases from 0.775 at G=1 to 0.799 at G=20 on Terminal-Bench. This reduces tie rates and improves calibration.

**2. Repeated Evaluation (K):** Averaging K independent evaluations acts as a Monte Carlo estimator whose variance shrinks as O(1/K). Accuracy rises from 74.7% at K=1 to 77.5% at K=16. A single-pass verifier (K=1) already matches a heavily ensembled judge (K=16).

**3. Criteria Decomposition (C):** A single monolithic rubric is replaced with an ensemble of sub-criteria. For code agents: Specification, Output, and Errors. Any single criterion achieves 75.2–76.4% accuracy; the ensemble reaches 78.3%.

#### Probabilistic Pivot Tournament (PPT)

A cost-efficient ranking algorithm reducing budget from O(N²) to O(Nk):

1. **Ring pass:** A random Hamiltonian cycle scores N adjacent pairs, canceling positional bias since every candidate appears once in "A" slot and once in "B."
2. **Pivot selection:** Top-k candidates by ring-pass mean preference become pivots.
3. **Pivot rounds:** Every non-pivot vs. pivot and pivot vs. pivot pair is scored, concentrating budget on uncertain top candidates.

The candidate with the highest count-normalized win score is returned.

### 3. Benchmark Results

#### Terminal-Bench V2 (86.5% — SOTA)
- Shell-based agent tasks with multi-step reasoning
- Capy scaffold, GPT-5.5 proposals (N=5), Gemini 2.5 Flash as verifier
- Pass@1: 83.1%, oracle Pass@5: 92.1%
- Surpasses GPT-5.5+NexAU-AHE (84.7%), Opus 4.7+WOZCODE (80.2%), Gemini 3.1 Pro+TongAgents (80.2%)
- Generalizes across harnesses: Terminus-Kira (79.4%) and Terminus-2 (71.2%)

#### SWE-Bench Verified (78.2% — SOTA)
- 500 real-world GitHub issues requiring patches
- mini-swe-agent, heterogeneous pool (N=3) from Opus 4.5, Gemini 3 Flash, MiniMax M2.5
- Mean Pass@1: 76.1%, oracle Pass@3: 84.4%
- Outperforms individual models (Opus 4.5 at 76.8%, Gemini 3 Flash at 75.8%, M2.5 at 75.8%)

#### RoboRewardBench (87.4% — SOTA)
- Robotic manipulation trajectory preference prediction with video inputs
- Qwen 3.6 35B as base VLM verifier, G=20, K=8
- Outperforms trained models: RoboReward-8B (81.4%), Robometer-4B (78.8%), TOPReward (74.7%), discrete LLM-as-a-Judge (70.8%)
- MAE reduced from 1.11 to 0.72 vs. human annotations

#### MedAgentBench (73.3% — SOTA)
- Medical tasks in simulated EHR environment with tool use
- Claude Opus 4.8 proposals (N=5), Pass@1: 70.2%
- Beats Opus 4.8 (70.2%), Gemini 3.5 Flash (66.3%), GPT-5.5 (65.1%)

### 4. Task Progress Estimation

The verifier's continuous score correlates strongly with chronological task progress (Value-Order Correlation — Spearman rank correlation between step index and verifier score):

- Code generation (Terminal-Bench V2): Successful trajectories achieve VOC of 0.848; failed trajectories score 0.769 (0.08 gap)
- Robotics (RoboRewardBench): VOC of 0.966 — substantially exceeding RoboReward-8B (0.877), Robometer-4B (0.780), TOPReward (0.565)

A concrete example: the successful MNIST inference trajectory's verifier scores rise monotonically through steps (Read model.py → Install g++ → Install CPU-only torch → Update hidden_dim → DONE), while the failed trajectory that unnecessarily installs torchvision and exhausts disk space receives consistently low scores.

### 5. TurboAgent Extension

A drop-in extension for Claude Code and OpenAI-API compatible clients. Operates as an inference-time proxy sitting between client and LLM provider, dispatching N parallel candidate trajectories and selecting the best via PPT. Also provides a web-based interface for visualizing verifier outputs and real-time agent progress monitoring.

### 6. Dense Reward for RL

#### Off-policy RL (DSRL-SAC)
- Fine-tunes π0 VLA model on LIBERO ketchup task
- Relabels rollouts with shaped reward: r_t = r_t^env + λ·ρ_t
- Achieves ~1.8× higher sample efficiency than sparse reward baselines
- Reaches higher final success rate (0.76 vs. 0.69)

#### On-policy RL (GRPO)
- Fine-tunes Qwen3-8B on MATH with GRPO
- Adds reasoning trace preference score to correctness and format rewards
- Mitigates collapse of group-relative advantage when all sampled responses produce incorrect final answers
- Achieves ~1.1× sample efficiency gain (~10% reduction in optimizer steps)

### 7. Key Findings

#### Verification Scaling
Verification accuracy consistently improves along all three dimensions simultaneously, rising from 73.1% at baseline when all three are combined. The axes are complementary: granularity sharpens individual estimators, repeated evaluation averages out noise, and criteria decomposition reduces prompt bias.

#### Judge vs. Verifier Comparison
On query-optimize case study over 100 repeated evaluations:
- Discrete judge (G=5): 12/100 correct rankings, 88/100 ties, 0/100 wrong
- Continuous verifier (G=5): 69/100 correct, 0 ties, 31/100 wrong
- Continuous verifier (G=20): 77/100 correct, 0 ties, 23/100 wrong

The verifier eliminates ties entirely and substantially improves discrimination.

#### Compatibility with Frontier Models
For models like GPT-5.5 and Claude Opus 4.7 that don't expose logprobs, a two-stage pipeline: closed model emits free-form reasoning, then an open verifier (Gemini 2.5 Flash) reads logprobs from scoring tokens. Recovers +5.2-point accuracy gain over discrete scoring at K=1 and eliminates the 10.9% tie rate.

#### PPT Ablation
On 20-candidate pools (89 Terminal-Bench V2 tasks):
- PPT with k=3 surpasses V1 baseline (66.17% accuracy with 4,723 pairs)
- PPT with k=9 reaches 67.13% with 9,630 pairs, approaching full round-robin (67.42% with 13,111 pairs) at ~73% of the budget

### 8. Related Work

Situated at intersection of: Test-Time Scaling, LLM-as-a-Judge, and Verifiable Reward. Differs from trained reward models (PRMs, ORMs, robotic reward models) by being training-free and general-purpose.

### 9. Limitations (Appendix A)

1. Requires scoring-token logit access (mitigated by two-stage workaround with open models)
2. Criteria decomposition is hand-designed per domain rather than learned
3. RL experiments limited to single-turn settings; multi-turn RL with per-step rewards over long-horizon rollouts is future work

### 10. Conclusions

Verification is an underexplored scaling axis. LLM-as-a-Verifier delivers fine-grained feedback via logit expectation, enabling scaling across granularity, repeated evaluation, and criteria decomposition. SOTA across coding, robotics, and medical domains. Beyond ranking, the signal serves as a task progress estimator and dense reward for RL, improving sample efficiency in both SAC and GRPO settings. The framework is training-free, plug-and-play, and applied identically across all domains without per-domain fine-tuning.
