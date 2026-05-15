---
title: "How to Write a Good Spec for Agents"
url: https://addyosmani.com/blog/good-spec/
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# How to Write a Good Spec for AI Agents - Addy Osmani

Framework for writing effective specifications that guide AI coding agents productively. Emphasizes clarity, modularity, and iterative refinement.

## Five Core Principles

### 1. Start with High-Level Vision, Let AI Draft Details
Begin with a concise product brief and request the AI to expand it into a detailed spec. Use Plan Mode (read-only analysis) before execution.

### 2. Structure Like a Professional PRD
Six essential areas identified from 2,500+ agent configuration files:
1. Commands - executable commands with flags
2. Testing - framework, location, coverage expectations
3. Project structure - explicit directory organization
4. Code style - real examples over descriptions
5. Git workflow - branch naming, commit formats
6. Boundaries - what agents must never touch

Three-tier boundary system: Always do / Ask first / Never do.

### 3. Break Tasks into Modular Prompts
"Curse of instructions" research: model performance drops as instruction count increases. Create separate spec files for different domains. Run parallel agents on non-overlapping tasks.

### 4. Build Self-Checks, Constraints, and Domain Knowledge
Instruct agents to verify against spec after implementation. Use "LLM-as-a-Judge" for subjective criteria. Build conformance testing from specs. Include examples, not just descriptions.

### 5. Test, Iterate, and Evolve Continuously
Spec-writing is cyclical, not linear. Run tests after milestones. Update specs when discovering incompleteness. Track specs in version control alongside code.

## Common Pitfalls
- Vague prompts without concrete constraints
- Overlong contexts without hierarchical summarization
- Skipping human review of critical code paths
- "Lethal trifecta": speed + non-determinism + cost

## Key Distinction
"Vibe coding" (rapid exploration) differs fundamentally from "AI-assisted engineering" (disciplined, production-ready work requiring rigorous specs and testing).
