---
url: https://claude.dev/blog/how-we-made-claude-ai-faster/
title: "How We Made Claude AI Faster"
author: Anthropic engineering team (Issac, Sam, Shelley, et al.)
date_fetched: 2026-09-25
date_published: 2026-09 (September 2026)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

A field report from Anthropic: in a two-week August sprint, a Slack channel with Claude (via Claude Tag, running an internal Opus-5.5-class model) made claude.ai and the desktop app about 3x faster. Time to a typeable page at p75 went from 3.1s to 0.55s; over 3,000 changes merged with zero customer-facing incidents.

The method: pick the four user journeys covering 95% of activity, instrument them into thirteen comparable measurements, then let Claude hill-climb. The sprint's central lesson is that with Claude, **measurement is step one of the climb, not step zero** — as soon as the model has a number to beat, it can optimize, so the highest-leverage human work was finding more things to measure. Every benchmark had two jobs: something Claude could move in the lab, and a CI ratchet that could only ratchet down.

The scale is the story. Claude ran 150+ concurrent threads, put up 50–100 optimization PRs per thread, and up to 200+ changes landed on the busiest days. Quirky finds include UTF-16 strings slowing syntax highlighting (em dashes!), a `:root:has()` selector costing 24ms per DOM change, and half a million hidden `location.reload()`s a day. Safety rested on human-approved PRs, unit tests before optimizations, short-lived feature flags (~200 flags, over half cleaned up within the sprint), incremental rollouts, and dozens of visual guardrails protecting fragile wins like the static composer.

Humans were not idle; they supplied the three things the loop lacked: **Ambition** ("targets are not the stopping point"), **Taste** (ruling on user-perceptible tradeoffs with screenshots), and **Direction** (sequencing, closing threads at diminishing returns — one 900-line PR was gavelled down because "2ms per send is not worth the complexity"). The loop was productive, but explicitly not autonomous.
