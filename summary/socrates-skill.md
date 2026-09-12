---
url: https://github.com/bevibing/socrates-skill
title: "Socrates Skill"
author: Choi Wontak (RoundTable02)
date_fetched: 2026-08-01
date_published: 2026-04-02
topics:
  - claude-code
---

A Claude Code Agent Skill that implements the Socratic method: the agent reads the material silently, then guides the user to answers through progressive questioning — it never gives a direct answer. The entire skill is a single 73-line markdown file (no code, no dependencies, no state).

The skill operates as a triggered behavioral overlay: when the user mentions "Socrates," "socratic," or "소크라테스," the agent host injects the skill's instructions, replacing the default helpful-assistant stance with a Socratic tutor. It follows a five-phase workflow: read and understand the subject silently, assess the user's current understanding with an opening question, guide through escalating questions (Clarifying → Probing → Connecting → Counter → Hypothetical), respond adaptively based on whether the user is correct/wrong/stuck/asking-directly, and confirm understanding through summarization.

The sharpest design move is the anti-pattern list — five specific failure modes the agent must avoid (e.g., stating the answer then asking "do you understand?", giving hints so obvious they're effectively answers). This constructive negativity is more reliable than positive specification because boundary violations are easier for the model to detect than insufficient compliance.

Key limitations: no memory across sessions, no adaptation for learner frustration, no domain-specific question calibration, and the "never answer" constraint tends to erode under long conversations. The skill trades depth for universality — the same five question types apply to code, documents, or any readable content.

Distributed via `npx skills add RoundTable02/socrates-skill`. MIT licensed.
