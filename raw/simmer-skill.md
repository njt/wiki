---
url: https://2389.ai/posts/simmer-skill/
title: "Simmer: A Self Honing Skill"
author: Michael Sugimura
date_fetched: 2026-05-15
date_published: 2026-03-13
---

# Simmer: A Self Honing Skill

Michael Sugimura, March 13, 2026. 2 min read.

Simmer is a Claude Code skill for iterative artifact refinement. It draws inspiration from Berkeley researchers' work on "Actionable Side Information (ASI)" — applying RL-style feedback loops to text tasks where evaluation and prioritized, actionable feedback are possible.

The skill works by having an agent generate content, judge it against defined criteria, feed prioritized fixes back, and repeat — applicable to anything text-shaped (adventure hooks, pitch emails, API specs, blog posts).

The authors tested Simmer by using it to refine itself via an inner/outer agentic loop: the outer loop evaluated and improved the skill definition; the inner loop spun up three agents per skill version running test tasks independently.

## Key Lessons

1. **Judges need calibration or they inflate:** Early runs saw all scores at 9.2 despite poor output quality. The fix: "giving the judge the seed artifact and its iteration-0 scores as permanent context every round" plus explicit score-level anchors.

2. **The skill improved faster than the artifacts did:** Artifacts were decent from run one; the real gains came from making instructions unambiguous. "Replace the instruction with an explicit contract" resolved inconsistencies like varying iteration counts and table schemas across agents.

## Why It Works

Unlike traditional ML requiring thousands of random iterations, backbone LLMs start with "massive pretrained competence." The model already knows what good output looks like — it just needs targeted feedback on what's missing from a specific artifact. ASI means "pointed feedback plus a capable agent means you converge in three to five rounds instead of three thousand."

## Key Concepts

- **Actionable Side Information (ASI):** Feedback focused on what to improve next, precise enough for the generator to act on
- **Self-honing / meta-iteration:** Using the skill to improve its own definition
- **Inner/outer agentic loops:** Multi-agent evaluation architecture
- **Judge calibration:** Preventing score drift with anchoring context
- **Explicit contracts vs. ambiguous instructions:** Reducing agent variance through specificity
- **Convergence speed:** Leveraging pretrained competence for rapid refinement (3–5 rounds vs. thousands)

Installation: `/plugin marketplace add 2389-research/claude-plugins` then `/simmer`

Tags: agents, self-improvement, claude-code, agent-skill, skills, reinforcement-learning, refinement, iteration, quality, multi-agent
