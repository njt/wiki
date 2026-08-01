---
url: https://www.netlify.com/blog/how-we-measure-netlify-agent-experience/
title: "How we measure Netlify's Agent Experience with AXIS"
author: Netlify
date_fetched: 2026-07-08
date_published: 2026-07-08
---

AXIS is an open-source scoring framework for Agent Experience (AX) — think Lighthouse, but for how well a platform or API serves AI agents rather than humans. Users supply a scenario (a JSON file with a prompt and scoring rubric), point AXIS at an agent, and it scores the run across four dimensions: Goal achievement, Service, Environment, and Agent. The output is a 0–100 score and an HTML report. It supports 22 agents natively.

Netlify ran three scenarios against their own platform using Claude Code and Codex, testing both "no-context" and "with-skill" conditions. The aggregate AXIS score for Netlify Agent Runners was 84/100. Skills — agent-side context files — boosted scores by an average of 26 points, and every with-skill run reduced both time and token cost compared to the no-context baseline. The most dramatic case: delegate-local-wip with Codex jumped from 63 to 98 while nearly halving tokens.

The article frames a broader vision: a context pipeline that automatically derives, tests, and publishes agent context from existing documentation repos. In this model, AXIS runs as CI on every PR, and score regressions are treated like broken tests. The goal is that new docs can't ship without a deliberate decision about their agent context.

The piece notes that interest in Agent Experience surged after a Queen's University study found 97% of MCP tool descriptions had quality issues — not a protocol flaw, but a symptom of no one designing with the agent as the end user. Auth0 is cited as a founding contributor to AXIS.
