---
url: https://newsletter.posthog.com/p/decision-models
title: "Decision Models: What We've Learned"
author: Jina Yoon
date_fetched: 2026-11-20
date_published: 2026-11-20
topics:
  - agent-architecture
  - ai-research-and-models
---

PostHog's field report on decision models — small models like Jev that answer structured questions with a typed answer plus a calibrated probability ("Yes, 0.99") instead of free-form prose. Several uses are in production, and the piece is a practitioner's taxonomy of when to use them and when not to.

**Gate expensive LLM calls.** In Replay Vision, two narrow yes/no questions ("Does this name a real person?", "Does this show contact details or an ID?") scrub PII from session summaries; 98.7% of summaries skip the expensive LLM call entirely. Thresholds are set by which mistake costs more: 0.4 for PII (false negative = stored personal data; false positive = one wasted call), 0.85+ for the Slack bot's decide-whether-to-speak check (false positive = being annoying in channels).

**Model routing.** A single `choice` question ("pick the cheapest option that will most likely finish the task correctly on the first try") routes Slack-bot requests to the cheapest capable model, with each option carrying a note on what it's good for.

**Face-value checks.** A decision model can't rank search results from scratch, but after embeddings produce a candidate list, a single question per result ("Does this summary match what the search looks for?") reranks them — a 25% quality improvement on 20 test queries.

**Where they fail.** Thresholds don't transfer across model versions (v1→v2 forced recalibration from hand-labeled data, hence a model-agnostic internal AI gateway). Multi-step chained decision-model jobs were operationally disastrous (an hourly job flooding the model server, blocking other products). Trying to rank PR-review-agent findings got AUC 0.52 — inconclusive — because decision models can't absorb large context. Hybrid models like Jeeves (reasons before deciding when unsure) bridge this gap; Jeeves is now PostHog's default.

Takeaway formula: decision models judge what's in front of them in one pass — evaluators for inputs too fuzzy for a regex and too simple for an LLM.
