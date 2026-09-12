---
url: https://arxiv.org/html/2510.22977v2
title: "The Reasoning Trap: How Enhancing LLM Reasoning Amplifies Tool Hallucination"
author: "Chenlong Yin, Zeyang Sha, Shiwen Cui, Changhua Meng, Zechao Li"
date_fetched: 2026-07-05
date_published: 2026-04-17
topics:
  - ai-research-and-models
---

# The Reasoning Trap: How Enhancing LLM Reasoning Amplifies Tool Hallucination

**Authors:** Chenlong Yin (Penn State), Zeyang Sha (Nanjing Univ. of Science & Technology, corresponding), Shiwen Cui, Changhua Meng (Independent), Zechao Li (Nanjing Univ. of Science & Technology)
**arXiv:** 2510.22977v2 [cs.LG], 17 Apr 2026
**License:** CC BY 4.0

## Abstract

The paper investigates a paradox where stronger reasoning in LLMs correlates with increased hallucination, specifically tool hallucination in agentic settings. The authors introduce SimpleToolHalluBench, a diagnostic benchmark, and demonstrate through controlled experiments that "gains in task performance are consistently accompanied by higher tool hallucination rates" across RL, distillation, and toggleable reasoning modes. Training on non-tool tasks (e.g., math) still amplifies tool hallucination. Mitigation strategies reveal a "fundamental reliability–capability trade-off: reducing hallucination unavoidably degrades utility."

## 1. Introduction

The authors describe the evolution of LLMs from text generators into Agents that combine internal deliberation with external tool calls. They coin **tool hallucination** — when models "fabricate non-existent tools or misappropriate available but irrelevant tools."

Three research questions guide the work:
- **RQ1:** Does enhancing reasoning amplify tool hallucination?
- **RQ2:** What are the underlying mechanistic drivers?
- **RQ3:** To what extent can tool hallucination be effectively mitigated?

**Contributions:** (1) SimpleToolHalluBench, a lightweight diagnostic benchmark; (2) first experimental and mechanistic evidence that reasoning-focused RL amplifies tool hallucination across training methods and model families; (3) demonstration of a reliability-capability trade-off in mitigation.

## 2. Related Work

The paper situates itself in three literature streams:

- **LLMs as Tool-Using Agents:** Building on CoT prompting (Wei et al., 2022), ReAct (Yao et al., 2023), and Toolformer (Schick et al., 2024) — systems that interleave reasoning with tool calls.
- **Reinforcement Learning for Reasoning:** From PPO-style process supervision to GRPO-style outcome-level optimization (Shao et al., 2024; Guo et al., 2025), increasingly used for agentic reasoning frameworks like Search-R1, ReCall, and ToolRL.
- **Hallucination in LLMs:** General hallucination literature (Zhang et al., 2025) plus the specific sub-problem of tool hallucination, with benchmarks like ToolBeHonest (Zhang et al., 2024a).

## 3. SimpleToolHalluBench: A Benchmark for Tool Hallucination

The benchmark probes whether "agents can reliably abstain from tool use when no appropriate tools are available."

### 3.1 Benchmark Design

Two fundamental scenarios:

- **No-Tool-Available Task (NTA):** System prompt provides no tools; user query explicitly requires external tool invocation. Measures whether agents hallucinate non-existent tools.
- **Distractor-Tool Task (DT):** System prompt includes an irrelevant distractor tool; query requires a different, unprovided tool. Tests whether agents misuse distractor tools or hallucinate better ones.

The benchmark uses 296 tools from AgentSafetyBench (Zhang et al., 2024b), with queries generated via ChatGPT-4o. Crucially, queries cannot be resolved through internal knowledge — they are "absolutely impossible to complete correctly" in both settings.

Hallucination rates are computed separately per task as the fraction of responses flagged as hallucinated by an LLM-as-judge.

## 4. Tool Hallucination in Reasoning RL

Four sequential experiments isolate the root cause.

### 4.1 The Side-Effects of Tool-Specific Reasoning RL

The authors replicate ReCall (Chen et al., 2025), a GRPO-based agentic reasoning framework, using Qwen2.5-7B-Instruct trained on the SynTool dataset. Checkpoints are saved every 100 steps.

Results show a clear trade-off: SynTool validation reward steadily improves, but hallucination rates on both NTA and DT tasks simultaneously and substantially increase. As the authors state, agents "become over-eager to apply this behavior, even in contexts where tools are missing."

### 4.2 Non-Agentic Reasoning RL Can Also Drive Tool Hallucination

To test whether this is merely overfitting to tool-use patterns, the authors train on **GSM8K** — pure math problems with no tool involvement whatsoever — using GRPO. Despite no tool-related supervision, hallucination rates on SimpleToolHalluBench consistently rise as math reasoning accuracy improves.

The authors conclude: "The increase of tool hallucination cannot be fully attributed to overfitting on tool-use data." Reinforcing chain-of-thought reasoning appears to instill a general tendency to fill gaps with plausible but unsupported content.

### 4.3 Generalizing the Impact of Reasoning on Tool Hallucination

Beyond RL, the authors test whether other reasoning enhancement methods produce similar effects:

- **Qwen2.5-7B vs. R1-Distill-Qwen-7B** — distillation inherits hallucination tendencies
- **Qwen3-8B and Qwen3-32B with "Thinking" mode toggled on/off**

**Key results (Table 1):**

| Model | Config | R_NTA | R_DT |
|-------|--------|-------|------|
| Qwen2.5-7B | Instruct | 34.8 | 54.7 |
| | R1-Distill | 74.3 | 78.7 |
| Qwen3-8B | Think Off | 4.1 | 36.2 |
| | Think On | 5.4 | 56.8 |
| Qwen3-32B | Think Off | 5.1 | 46.6 |
| | Think On | 8.8 | 50.7 |

The pattern holds across model families and scales — reasoning-enhanced versions consistently hallucinate more. **Appendix B** extends results to DeepSeek-V3 vs. R1 (671B) and Kimi-K2, confirming the trend at the largest scales.

### 4.4 Isolating Reasoning as the Driving Factor

Two controlled experiments rule out alternative explanations.

#### 4.4.1 Ablating the Reasoning from RL Training

Training on SynTool under two regimes: direct tool-use RL (no <think> block) vs. think-then-act RL (standard ReCall). **Table 2** shows:

- Direct tool-use RL: R_NTA rises from 34.8 to 41.4 (moderate)
- Think-then-act RL: R_NTA jumps to **90.2** (dramatic)

Since the two regimes share everything except the reasoning step, "the sharp gap indicates that the observed amplification is most closely associated with the *reasoning step itself*."

#### 4.4.2 Ruling Out General Instruction-Following Degradation

Evaluating ReCall-7B on IFEval, ComplexBench, and BFCL Multi-Turn:

- Instruction-following remains stable (IFEval drops only 2.6%)
- Tool-calling competence *improves* substantially (BFCL +9.9%)
- Yet tool hallucination surges (R_NTA: 34.8% → 90.2%; R_DT: 54.7% → 100.0%)

This demonstrates tool hallucination is a **distinct failure mode** not captured by existing benchmarks.

## 5. Mechanistic Analysis

### 5.1 Representation Collapse: Reasoning RL Destabilizes Tool Pathways

Using Centered Kernel Alignment (CKA), the authors compare representations of the original Qwen2.5-7B-Instruct vs. the GRPO-trained version (trained on GSM8K math). CKA measures similarity between neural representations on a scale from 0 (completely dissimilar) to 1 (identical).

**Key finding (Figure 4):** In-distribution (math) representations remain highly stable (CKA > 0.9 across layers), while tool-related representations show "dramatic collapse" with CKA below 0.75 in early and middle layers. This asymmetry indicates that Reasoning RL "substantially reshapes the model's representation space" on tool-related inputs while preserving training-domain pathways.

### 5.2 Localizing Activation Differences

For each architectural component per layer (attention output, MLP output, residual stream at two points), the authors train linear classifiers to distinguish correct vs. hallucinated responses, defining a **discrimination score** as accuracy gain over random baseline (0.5).

**Results (Figure 5):** Residual stream components, especially from layer 20 onward, exhibit discrimination scores exceeding 0.14, far higher than attention (avg. 0.02) and MLP (avg. 0.04) outputs. The authors interpret this as consistent with the residual stream being "the primary pathway for accumulating information" — small differences compound during propagation, manifesting in late layers as distinct activation patterns.

**Appendix E** provides additional CKA analyses at the module level (attention vs. MLP vs. residual stream) and cross-domain comparisons (SynTool vs. GSM8K), showing that MLP representations drift more than attention, and that even SynTool in-distribution inputs undergo substantial representational shift.

## 6. Is There a Free Lunch in Mitigating Tool Hallucination?

### 6.1 Methodology

Two mitigation approaches tested on the ReCall-7B model (which shows heightened hallucination post-RL):

- **Prompt Engineering:** Adding "You must not use any tools that are not explicitly provided to you" to the system prompt.
- **Direct Preference Optimization (DPO):** Training on preference data where: (1) when tools are unavailable, chosen = honest abstention, rejected = fabricated tool calls; (2) when tools are available, chosen = correct invocation, rejected = needless refusal to prevent over-passivity.

### 6.2 Results and Analysis

**Table 4:**

| Method | R_NTA | R_DT | Reward |
|--------|-------|------|--------|
| ReCall-7B | 90.2 | 100.0 | 0.45 |
| + Prompt Eng. | 87.5 | 98.9 | 0.44 |
| + DPO | 55.8 | 71.4 | 0.34 |

Prompt engineering yields only marginal gains — "the model largely ignores the directive." DPO substantially reduces hallucination but at the cost of a significant drop in validation reward (0.45 → 0.34), meaning the agent "becomes less effective at proficiently using tools even in appropriate scenarios."

## 7. Conclusion and Outlook

The paper uncovers a fundamental paradox: techniques enhancing reasoning are "consistently accompanied by reduced tool-use reliability and increased hallucination." The effect is most closely associated with reasoning itself rather than RL training in general. The mechanistic analysis shows reasoning RL induces "disproportionately larger representational shifts on tool-related inputs," with late-layer residual streams as the locus where correct and hallucinated responses become linearly separable. A severe reliability-capability trade-off exists — DPO improves honesty but degrades utility. The authors call for objectives that "explicitly co-optimize for confidence calibration and abstention."

### Limitations

The benchmark focuses on single-step tool invocation (not multi-step tool chains); the mechanistic analysis is not yet a complete causal account; and mitigations explored are limited to prompt engineering and DPO. Other techniques like process supervision, constitutional AI, or novel reward shaping remain unexplored.

### Ethical Considerations

The paper highlights reliability risks from reasoning optimization, noting that "stronger reasoning can increase the tendency to fabricate or misuse tools." The authors recommend additional protections — "explicit abstention mechanisms, human oversight, and stricter validation of tool calls" — for high-stakes deployments.

## Appendix Summaries

- **Appendix A:** Construction details of SimpleToolHalluBench (349 tools→296 after quality filtering), system prompts for reasoning and non-reasoning models, mitigation experiment prompts, query examples with labeled responses, and LLM-as-Judge evaluation prompts using DeepSeek-R1.
- **Appendix B:** Extended results on Qwen3-4B, Qwen3-235B, DeepSeek-V3 vs. R1 (671B), and Kimi-K2 — confirming the "consistent degradation across scales and families."
- **Appendix C:** Algorithmic details of GRPO (group-relative advantage computation, clipped policy gradient with KL penalty) and DPO (pairwise preference optimization, gradient intuition, and the paper's specific preference construction).
- **Appendix D:** Details of ReCall framework — training only on SynTool to isolate effects, using GRPO with vLLM/SGLang inference stack, periodic checkpointing.
- **Appendix E:** Module-level CKA (attention vs. MLP vs. residual stream — MLP drifts more than attention) and cross-domain CKA (SynTool vs. GSM8K — both domains undergo substantial representational drift).
