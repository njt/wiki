---
url: https://cognition.com/blog/devin-fusion
title: "Devin Fusion: Frontier Performance at 35% Lower Cost"
author: The Cognition Team
date_fetched: 2026-07-18
date_published: 2026-06-29
---

Cognition's blog post introduces **Devin Fusion**, a multi-model agent harness designed to route work across frontier and cheaper models. The core claim is that it delivers "frontier and Fable 5-level performance at 35% lower cost" on their FrontierCode coding benchmark.

## People & Products Mentioned

- **Devin** — Cognition's AI software engineering agent
- **Fable 5** — A model (access suspended June 12, 2026 per U.S. government directive; referenced via [anthropic.com/news/fable-mythos-access](https://www.anthropic.com/news/fable-mythos-access))
- **Opus 4.8** — Reference frontier model
- **GPT-5.5** — Reference frontier model
- **GLM-5.2** — Reference frontier model
- **Walden** — Contact person (email: walden@cognition.ai)
- **Cognition** — The company (also Cognition AI Labs)

## Technical Details

### The Sidekick Approach
The architecture runs two parallel agents: a frontier model as the "main agent" and a more cost-effective "sidekick" model. Both maintain their own persistent, cached contexts. The main agent is instructed to "take minimal actions" and "only read what is absolutely necessary," delegating work to the sidekick by default while retaining decision authority over planning, ambiguity interpretation, and final review.

Key benefits outlined:

1. Retains real frontier intelligence rather than benchmark-optimized performance.
2. Generalizes beyond single-prompt tasks — routing can shift mid-session.
3. Avoids costly cache misses by maintaining separate cached contexts for each agent.

### Dynamic Mid-Session Routing
Lightweight classifiers run during task execution to signal when to switch models. The system switches models during **context compaction** (which would trigger a cache miss anyway), making model switching effectively free. This allows upgrading the sidekick model mid-task without additional cache penalty.

### Benchmarks (FrontierCode Extended)

| Configuration | Score | Avg Cost/ Task |
|---|---|---|
| Fusion + Fable 5 | 57.6 | $3.00 |
| Fable 5 (medium) | 57.0 | $5.12 |
| Opus 4.8 (high) | 48.8 | $3.24 |
| Fusion | 47.9 | $2.38 |
| GPT-5.5 (high) | 44.8 | $3.64 |
| GLM-5.2 | 43.0 | $2.70 |

Fusion without Fable 5 achieved a **35% cost improvement** vs. GPT-5.5/Opus 4.8. Fusion + Fable 5 achieved **41% cost reduction** vs. pure Fable 5.

### Worked Examples (Sidekick Impact)

1. **Modernize `search.js` to ES6** — Cost saved 62% ($3.55→$1.37); quality maintained. "The cost was in the tests, not the code."
2. **Rip out OpenTracing in Mattermost** — Cost saved 32% ($3.80→$2.57); near-identical score (97 vs. 98).
3. **JSON-Schema `oneOf` Python model generation** — Cost saved 38% ($5.08→$3.13); same partial result.
4. **Team selector in search bar (hard React/Redux)** — Cost saved 28% but score dropped from 54 to 27. "When the judgment is the deliverable, delegating it backfires."
5. **LangChain4j WebSocket MCP into Quarkus** — Cost saved 25% ($5.25→$3.93); score increased from 69 to 81.

### Sanity Check (Internal Usage)
88% of merged PRs from internal Cognition users were "driven entirely by the automated Fusion router."

## Broader Argument
The post argues that "the age of using one model for all of your work is coming to an end," citing rising frontier model costs and the growing diversity of capable models. It analogizes that you would not use a Lamborghini for a grocery trip. Multi-model harnesses also capture relative strengths — e.g., some models excel at UI testing while others identify bugs in PRs.

The post ends by inviting readers to try Devin Fusion in preview at [app.devin.ai/signup](https://app.devin.ai/signup?source=fusion_blog) and to consider [careers at Cognition](https://cognition.com/careers).
