---
url: https://gist.github.com/8f638d27df4504ece90af0623ff6ce1a
title: "Lessons from building Vercel v0 and the d0 agent"
author: Malte Ubl (CTO, Vercel)
via: The Pragmatic Engineer (Gergely Orosz)
date_fetched: 2026-07-04
date_published: 2026-05-18
event: The Pragmatic Summit, San Francisco, February 11, 2026
youtube_url: https://www.youtube.com/watch?v=_f2WpsmW76Y
transcription_date: 2026-05-18
tags: [agent-design, simplicity, coding-agents, organizational-design, vercel]
topics:
  - agent-architecture
---

# Summary

Malte Ubl, CTO of Vercel, interviewed at The Pragmatic Summit about building Vercel's internal data agent Dzero and the V0 product agent. The central thesis: "building agents is actually extremely easy and you don't need to buy them."

## Dzero (Internal Data Agent)

A text-to-SQL engine in Slack. Started as many tools in a loop, rebuilt as a two-tool agent (bash + Execute SQL) — about 50 lines of code. Reads a YAML file describing every Snowflake column in plain English, then uses grep/tail to explore semantics and writes SQL.

**Key architectural insight:** As models get smarter, simpler agents work better. Complex tool loops become unnecessary. The YAML semantic layer is the durable artifact — it describes columns in plain English that the agent can grep through.

## V0 Evolution

Evolved through four user bases driven by model capability leaps:
1. Frontend engineers (early models, Tailwind CSS workaround)
2. Backend engineers (could fix V0's mistakes)
3. Full-stack builder for non-engineers (Sonnet 3.5 pivot)
4. Tech-adjacent roles (PMs, designers, business people) building internal tools

**The "Tailwind moment":** Early models couldn't handle separate CSS files. Instructing them to use Tailwind (inline styles) succeeded because models could reason inline. This made V0 viable in 2023.

**Framing as coding tasks:** Models excel at coding because of training data. Non-coding problems (text-to-SQL) perform better when structured as coding tasks — "you get disproportional good results."

## Organizational Practices

- **Optimistic locking:** No approval gates. Anyone can ship but must announce intent; the organization can veto. Shifts responsibility to vetoers.
- **Speed ≠ unreliability:** Control plane ships on every push to main; serving systems ship once daily. Autonomous regions with wave-based rollouts.
- **Unlimited tokens for developers:** It's in the job description. Cost is trivial versus productivity gains.
- **Solo exploration first:** Have one person find product-market fit before allocating a team. "Teams just, I don't know, make things go slower."

## Engineering Career Impact

- Senior ICs benefit most — they "now have just more minions"
- Junior engineers thrive as digital natives
- The middle is uncertain
- "The job looks much more like management than like IC work"

## Automation at Scale

- 87% of support intake automated — support agents now "have a much better job" handling only hard problems
- Open-sourced a sales lead qualification agent
- ~750 people at Vercel, CEO and speaker joke about capping at 1,024

## Key Metaphors

- **"Software is free, like a free puppy":** Marginal cost of creation approaches zero, but maintenance is the real burden
- **Mainframe-era analogy:** The 1960s shift eliminated rooms of human "computers" but ultimately made everyone richer
- **Self-driving infrastructure:** Agents participating in DevOps and application management — still nascent

## Full Transcript

The complete 34-minute transcript is available at the gist URL above. The interview was conducted by Gergely Orosz (The Pragmatic Engineer) at The Pragmatic Summit, February 11, 2026.
