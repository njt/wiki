---
url: https://www.oreilly.com/radar/the-agentic-data-science-playbook/
title: "The Agentic Data Science Playbook"
author: Hugo Bowne-Anderson, Eric Ma (Vanishing Gradients, republished on O'Reilly Radar)
date_fetched: 2026-10-03
date_published: 2026-09 (approximate; republished ahead of the Oct 6 course cohort)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

A playbook for doing data science *with* agents rather than through them. The authors open with a deliberately sabotaged experiment: Claude Opus 5.0 asked to "build a fraud detector" on a disguised version of the Elliptic Bitcoin graph dataset committed textbook errors — random splits across time steps (leaking the future into training) and use of a feature the authors planted as a label proxy. With proper guidance (temporal holdout, leakage removal, intended-use context, subgroup reporting), the same model's F1 fell from an illusory 0.87 to an honest 0.70, and recall on the operationally important high-degree nodes was only 0.21.

From this the article derives five practices: frame the investigation (an analytical brief specifying question, intended use, and what the agent should escalate rather than how to implement it); equip the agent (runtime, sandboxed workspace layout, skills encoding methodological rules like feature-availability checks, data documentation); organize the work (bounded autoresearcher loops against a frozen evaluator for predictive work; parallel analyses over competing assumptions for causal work); review independently (a fresh adversarial reviewer agent, kept blind to the investigator's path, plus human judgment); and convert reviewed failures into reusable skills and evals rather than leaving lessons in chat transcripts.

The framing is that the agent's two central responsibilities are **specification and verification**, with the data scientist remaining accountable for judging the question and the evidence. Concrete artifacts throughout: a task-prompt template, an experiment contract, a project directory layout with test data kept outside the agent's reach, and a lesson-to-skill conversion schema. The article closes with the multi-user problem — OpenAI's internal data agent and Meta's Analytics Agent as examples where analyst memory must become shared infrastructure.
