---
title: "LLM Evals"
url: https://hamel.dev/blog/posts/evals-faq/
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - guardrails-and-feedback-loops
---

# LLM Evals: Everything You Need to Know

By Hamel Husain. Comprehensive guide drawn from questions asked in an AI evals course.

## Core Concept

LLM evaluations are systematic processes for testing AI application quality through trace analysis, error detection, and iterative improvement -- not just foundation model benchmarking.

## Key Arguments

### Error Analysis is Foundational
The most critical evaluation activity involves manually reviewing user interactions (traces) to identify failure patterns. This includes open coding (noting issues), axial coding (categorizing failures), and iterative refinement until theoretical saturation. "Error analysis is the most important activity in evals."

### Binary Over Likert Scales
Pass/fail evaluations force clearer decision-making than 1-5 ratings. Numeric scales introduce subjective inconsistency, require larger sample sizes for statistical significance, and encourage middle-value defaults.

### Avoid Generic Metrics
Pre-built evaluation metrics like BERTScore or "helpfulness" ratings rarely capture application-specific needs. "All you get from using these prefab evals is you don't know what they actually do."

### Custom Annotation Tools Matter
Building domain-specific review interfaces yields approximately 10x faster iteration than off-the-shelf solutions.

## Resource Allocation

- Expect 60-80% of development time on error analysis and evaluation
- A 70% eval pass rate often indicates more meaningful testing than 100%

## Process Recommendations

- Benevolent Dictator Approach: Single domain expert drives quality standards
- Minimum Viable Setup: Manual review of 20-50 outputs using a notebook or simple custom interface
- Synthetic Data Strategy: Use structured dimensions and two-step generation
- Sampling: outlier detection, user feedback signals, metric-based sorting, stratified sampling, embedding clustering

## Domain-Specific Insights

- RAG Systems: Separate retrieval evaluation (Recall@k, Precision@k) from generation evaluation
- Agentic Workflows: Use end-to-end task success metrics first, then diagnose step-level failures
- Multi-turn Conversations: Focus on first upstream failures; downstream errors often cascade

## Key Distinctions

- Guardrails vs. Evaluators: Guardrails are synchronous safety checks; evaluators run asynchronously post-generation
- CI/CD vs. Production: CI uses small curated datasets (100+ examples) with deterministic checks

## Critical Warnings

- Don't skip error analysis for infrastructure optimization
- Avoid outsourcing core error analysis -- this breaks the learning feedback loop
- Don't adopt eval-driven development (writing evaluators before implementation)
- Beware optimizing for high eval pass rates

## Automation

LLMs can accelerate pattern discovery and categorization but cannot replace initial manual trace review or ground-truth validation.
