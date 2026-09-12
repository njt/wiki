---
title: "Demystifying Evals for AI Agents"
url: https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - guardrails-and-feedback-loops
  - security-and-sandboxing
---

# Demystifying Evals for AI Agents - Article Summary

## Key Arguments

**The Core Problem**: AI agents' capabilities—autonomy, intelligence, flexibility—make them harder to evaluate than traditional LLMs. Without rigorous evaluations, teams get stuck in reactive loops, catching issues only after deployment affects users.

**Why Evals Matter**: Teams initially make progress through manual testing and intuition, but this breaks down at scale. Evals provide visibility into problems before production, enable confident model upgrades, and compound in value throughout an agent's lifecycle. As one passage notes: "More rigorous evaluation may even seem like overhead that slows down shipping. But after the early prototyping stages, once an agent is in production and has started scaling, building without evals starts to break down."

## Core Definitions

The article establishes essential terminology:
- **Task/Problem**: A single test with defined inputs and success criteria
- **Trial**: One attempt at a task (multiple trials are run for consistency)
- **Grader**: Logic that scores agent performance (code-based, model-based, or human)
- **Transcript/Trace**: Complete record of all outputs, tool calls, and interactions
- **Outcome**: Final environmental state (the "real" result, not just what was said)
- **Evaluation Harness**: Infrastructure running evals end-to-end
- **Agent Harness/Scaffold**: System enabling the model to act as an agent

## Three Grader Types

**Code-Based Graders**
- Methods: string matching, regex, static analysis, outcome verification, tool verification
- Strengths: fast, cheap, objective, reproducible
- Weaknesses: brittle to valid variations, limited nuance for subjective tasks

**Model-Based Graders**
- Methods: rubric scoring, natural language assertions, pairwise comparison
- Strengths: flexible, scalable, captures nuance, handles open-ended tasks
- Weaknesses: non-deterministic, expensive, requires human calibration

**Human Graders**
- Methods: SME review, crowdsourcing, spot-checking, A/B testing
- Strengths: gold standard quality, matches expert judgment
- Weaknesses: expensive, slow, requires access to experts

## Agent-Specific Evaluation Strategies

**Coding Agents**: Rely on deterministic test suites (does the code run? do tests pass?). Example benchmarks like SWE-bench Verified improved from 40% to >80% in one year. Also grade transcript quality using static analysis and model-based rubrics.

**Conversational Agents**: Combine verifiable outcomes with interaction quality rubrics. Often require a second LLM simulating the user. Success is multidimensional: ticket resolution, turn count constraints, and appropriate tone.

**Research Agents**: Face unique challenges—expert disagreement, shifting ground truth, open-ended outputs. Combine groundedness checks (claims supported by sources), coverage checks (key facts included), and source quality verification.

**Computer Use Agents**: Interact via screenshots and clicks like humans. Evaluation requires real/sandboxed environments. Balance token efficiency (DOM extraction) against latency (screenshot analysis).

## Non-Determinism Metrics

Two critical metrics address variability:

**pass@k**: Probability of at least one correct solution in k attempts. Relevant when one success matters (e.g., "pass@1" = first-try success rate).

**pass^k**: Probability that all k trials succeed. Matters for customer-facing agents where consistency is essential. If per-trial success is 75%, pass^3 ≈ 42%.

The article illustrates: "At k=1, they're identical (both equal the per-trial success rate). By k=10, they tell opposite stories: pass@k approaches 100% while pass^k falls to 0%."

## 8-Step Implementation Roadmap

**Step 0-3: Task Collection**
1. Start early with 20-50 simple tasks from real failures (not hundreds)
2. Convert manual tests and bug reports into test cases
3. Write unambiguous tasks with reference solutions; ensure two experts would reach the same verdict

**Step 4-5: Harness and Grader Design**
4. Build robust eval harnesses with stable, isolated environments (clean state between trials)
5. Design thoughtful graders—prefer deterministic where possible; avoid brittleness by grading outcomes not pathways; build in partial credit

**Step 6-8: Long-Term Maintenance**
6. Read transcripts regularly to verify graders work and failures seem fair
7. Monitor for "eval saturation" (100% pass rate = no improvement signal)
8. Establish dedicated eval teams with clear ownership; enable domain experts to contribute

## Notable Practical Insights

**Task Quality Matters Most**: The article warns that a 0% pass rate across 100 trials usually signals a broken task, not an incapable agent. "Everything the grader checks should be clear from the task description; agents shouldn't fail due to ambiguous specs."

**Avoid Over-Specification**: "There is a common instinct to check that agents followed very specific steps like a sequence of tool calls in the right order. We've found this approach too rigid and results in overly brittle tests, as agents regularly find valid approaches that eval designers didn't anticipate."

**Balance Is Critical**: Build evals that test both when behaviors should occur and shouldn't. One-sided evals create one-sided optimization.

**Eval Saturation Risk**: As frontier models improve, evals approach saturation. SWE-Bench started at 30% this year and is nearing >80%.

## Complementary Methods

| Method | Strengths | Timing |
|--------|-----------|--------|
| Automated evals | Fast iteration, reproducible, no user impact | Pre-launch, CI/CD |
| Production monitoring | Real behavior at scale, catches surprises | Post-launch |
| A/B testing | Measures actual user outcomes | With sufficient traffic |
| User feedback | Surfaces unanticipated problems | Ongoing |
| Transcript review | Builds intuition, catches subtle issues | Weekly sampling |
| Human studies | Gold-standard calibration | As needed |

## Real-World Examples

- **Claude Code**: Started with fast iteration on feedback, later added narrow evals (concision, file edits), then broader behavioral evals (over-engineering)
- **Descript**: Evolved from manual to LLM grading with human calibration, now runs separate quality and regression suites
- **Bolt**: Added evals after launch using static analysis, browser agents, and LLM judges—built a system in 3 months

## Key Conclusions

Teams investing early in evals accelerate development. The value compounds through failures becoming test cases, test cases preventing regressions, metrics replacing guesswork, and clear targets for improvement. The most effective approach combines automated evals for iteration speed with production monitoring for ground truth and periodic human review for calibration. "Read the transcripts!" is essential practice.
