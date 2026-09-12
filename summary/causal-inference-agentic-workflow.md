---
url: https://medium.com/netflix-techblog/a-human-augmenting-agentic-workflow-for-causal-inference-4623f0a9c5af
title: "A Human-Augmenting Agentic Workflow for Causal Inference"
author: Winston Chou, Adrien Alexandre, Lars Olds, Yi Zhang, Garrett Hagemann, Nathan Kallus
date_fetched: 2026-06-22
date_published: 2026-06-08
source: Netflix TechBlog
topics:
  - agent-architecture
  - guardrails-and-feedback-loops
---

## Summary

Netflix presents an agentic workflow for Observational Causal Inference (OCI) under unconfoundedness. The system combines three personas (Principal/human, Actor/executor, Critic/evaluator) in an actor-critic loop to produce publishable causal inference artifacts—plans, specs, plots, and notebooks—that are inspectable at every step. The workflow builds on Netflix's pre-existing OCI toolkit (doubly robust learning, target trial emulation philosophy) and adds AI scaffolding that forces the LLM to follow rigorous causal inference templates. The authors open-source oci-agent and demonstrate that their scaffolded approach systematically beats one-shot prompting on the 2016 ACIC competition datasets.

## Key Architecture

The workflow uses three personas:
- **Principal**: The human data scientist who provides the initial analysis plan, identifies confounders and threats to valid inference, and specifies tools and data model.
- **Actor**: The software persona that refines the plan into a data analysis spec, executes analysis using only specified tools, performs all four design diagnostics (covariate balance, overlap, placebo outcome, sensitivity to hidden confounders), and creates inspectable artifacts.
- **Critic**: The software persona that checks for blind spots (unmentioned confounders, misalignment between plan/spec/execution), assigns a credibility level, flags if the estimand differs from ATE, and suggests alternative measurement strategies.

The Actor and Critic operate in a loop. The Principal evaluates artifacts produced by this loop.

## Four Design Diagnostics

1. **Covariate balance**: Standardized mean difference of pre-treatment covariates between groups should be below 0.2 after weighting.
2. **Overlap**: Propensity scores should fall between 0.1 and 0.9.
3. **Placebo outcome**: "Treatment effects" on pre-treatment variables should not be significantly different from zero.
4. **Sensitivity to hidden confounders**: Findings contextualized by sensitivity to hypothetical omitted variables.

## Case Study: Early Adopter Bias

A baseline Claude Sonnet 4.6 without scaffolding produced a defensible regression. The agentic workflow with the same model yielded an estimate only 25% of the baseline. The issue: early adopter bias—first users of a new offering differ systematically from the general population. The Critic flagged poor overlap and a failed placebo test. The Actor applied Crump-style trimming (removing units with propensity scores outside [0.1, 0.9]), producing a substantially more credible estimate.

## Evaluation Results

Two evaluations on 2016 ACIC competition datasets:
1. **Statistical methodology**: Competitive against 44 ACIC competition methods, with low RMSE and well-calibrated ~95% confidence intervals. The diagnostic suite successfully separates good from bad estimates.
2. **LLM scaffolding**: With scaffolding, the LLM recovers ground truth in 9/10 datasets. Without scaffolding (just prompting Sonnet 4.6), results are "consistently wrong answers that are not at all correlated with ground truth."

## Open Source

oci-agent at github.com/Netflix-Skunkworks/oci-agent, implementing the workflow with a lightweight causal ML notebook using only open-source software (EconML).
