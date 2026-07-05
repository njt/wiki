---
url: https://www.spicytakes.org/feed
date_fetched: 2026-07-05
backfilled: true
---

### Better Models: Worse Tools

**Key Insight:** Post-training a model inside a forgiving, dominant harness like Claude Code creates strong schema priors that cause newer, smarter models to hallucinate tool fields when encountering alternative schemas—making 'better' models actively worse for third-party harness developers.

Armin investigates a regression in Pi's editor where Opus 4.8 and Sonnet 5 invent extra, invalid fields in tool call arguments—fields like `requireUnique`, `matchCase`, and `oldText2` that Pi rejects as schema violations. He traces the root cause to post-training: Claude Code's own harness silently forgives schema slop (unknown keys, parameter aliases, unicode repairs), so RL training never penalizes the behavior. Models develop strong priors toward Claude Code's flat edit-tool shape, making alternative schemas increasingly off-distribution. This has uncomfortable implications for third-party harness developers who can no longer treat tool schemas as neutral contracts.

9Tool schemas are not neutral, at least not on Anthropic models.


8Alternative tool schemas might not just be unfamiliar. They might be implicitly punished by post-training that optimizes for one particular, forgiving tool ecology.


7The SOTA models of the family are worse at this specific tool schema than their older siblings.
