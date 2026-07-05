---
url: https://martinfowler.com/articles/harness-engineering.html
title: "Harness Engineering for Coding Agent Users"
author: Birgitta Böckeler
date_fetched: 2026-05-14
date_published: 2026-04-02
---

# Harness Engineering for Coding Agent Users

**Author:** Birgitta Böckeler
**Published:** 02 April 2026
**Source:** martinfowler.com

## Overview

This article explores how to build confidence in AI-generated code through a systematic approach called harness engineering. Böckeler defines the harness as "everything in an AI agent except the model itself" and structures it around two complementary strategies: feedforward controls (guides) and feedback controls (sensors).

## Key Formula

**Agent = Model + Harness**

## Key Concepts

### Feedforward and Feedback Controls

- **Guides (feedforward controls):** "Anticipate the agent's behaviour and aim to steer it before it acts. Guides increase the probability that the agent creates good results in the first attempt."
- **Sensors (feedback controls):** "Observe after the agent acts and help it self-correct. Particularly powerful when they produce signals that are optimised for LLM consumption."

"Separately, you get either an agent that keeps repeating the same mistakes (feedback-only) or an agent that encodes rules but never finds out whether they worked (feed-forward-only)."

"A well-built outer harness serves two goals: it increases the probability that the agent gets it right in the first place, and it provides a feedback loop that self-corrects."

### Computational vs. Inferential

- **Computational:** "Deterministic and fast, run by the CPU. Tests, linters, type checkers, structural analysis. Run in milliseconds to seconds; results are reliable."
- **Inferential:** "Semantic analysis, AI code review, 'LLM as judge'. Typically run by a GPU or NPU. Slower and more expensive; results are more non-deterministic."

### Harnessability

"Not every codebase is equally amenable to harnessing." Determined by language features, architectural clarity, framework abstractions. Strongly typed languages, clear module boundaries, and structured frameworks inherently afford better agent governance.

### Ambient Affordances

"Structural properties of the environment itself that make it legible, navigable, and tractable to agents operating within it" (coined by Ned Letcher).

## Section Structure

1. Feedforward and Feedback
2. Computational vs Inferential
3. The steering loop
4. Timing: Keep quality left
5. Regulation categories
   - Maintainability harness
   - Architecture fitness harness
   - Behaviour harness
6. Harnessability
7. Harness templates
8. The role of the human
9. A starting point — and open questions

Sidebars: Metaphors only go so far; How does harness engineering relate to context engineering?; Ambient affordances; Ashby's Law

## Harness Categories Table

| Direction | Computational/Inferential | Example |
|-----------|--------------------------|---------|
| Coding conventions | feedforward/Inferential | AGENTS.md, Skills |
| Bootstrap instructions | feedforward/Both | Skill + script |
| Code mods | feedforward/Computational | OpenRewrite recipes |
| Structural tests | feedback/Computational | ArchUnit hooks |

## Three Regulation Categories

1. **Maintainability Harness:** Regulates internal code quality through existing tooling — the most mature category. Tests, linters, type checkers, structural analysis.

2. **Architecture Fitness Harness:** Enforces architectural characteristics and performance requirements via fitness functions. Commits to a topology to narrow the space.

3. **Behaviour Harness:** Addresses functional correctness — currently the most challenging dimension. AI-generated test suites aren't trustworthy enough as the sole feedback mechanism.

## Change Lifecycle Stages

- Pre-integration (linters, fast tests, basic review)
- Integration + pipeline (mutation testing, broader review)
- Continuous drift monitoring (dead code, coverage quality, dependencies)
- Runtime feedback (SLO monitoring, response quality sampling)

## The Steering Loop

Humans iteratively improve the harness by addressing recurring issues, potentially using AI itself to generate custom controls, structural tests, and documentation.

## The Human Role

"A coding agent has none of this: no social accountability, no aesthetic disgust at a 300-line function."

"A good harness should not necessarily aim to fully eliminate human input, but to direct it to where our input is most important."

Developers bring implicit experience — organizational awareness, technical judgment, and accountability — that harnesses attempt to externalize.

## Named Examples

- OpenAI's layered architecture enforced through custom linters and structural tests
- Stripe's "minions" approach using pre-push hooks and "blueprints" integrating feedback into agent workflows
- Thoughtworks teams tackling architecture drift with mixed computational and inferential sensors
- Tools: ArchUnit, OpenRewrite, Semgrep, ESLint, Dependabot, Spring, LSPs, MCP servers

## Open Questions

- "So overall, we still have a lot to do to figure out good harnesses for functional behaviour that increase our confidence enough to reduce supervision and manual testing"
- AI-generated test suites aren't trustworthy enough as sole feedback mechanism
- How to keep harnesses coherent as they grow with non-contradictory guides/sensors
- How far agents can make sensible trade-offs with conflicting signals
- Need for harness coverage/quality evaluation similar to code coverage
- Harness template versioning and contribution problems
- Managing non-deterministic guides/sensors that are harder to test

## Actionable Recommendations

- "Keep quality left" — distribute checks earlier in development timeline
- Use computational sensors (cheap, deterministic) on every change
- Reserve inferential sensors for expensive post-integration checks
- Prioritize harness building where "harness is most needed" in legacy systems
- Commit to topologies to reduce variety and make harnesses more achievable
- Direct human effort "to where our input is most important"

## Key Concept: Ashby's Law

Referenced in sidebar — the law of requisite variety: a controller must have at least as much variety as the system it regulates.
