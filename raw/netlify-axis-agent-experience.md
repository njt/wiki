---
url: https://www.netlify.com/blog/how-we-measure-netlify-agent-experience/
title: "How we measure Netlify's Agent Experience with AXIS"
author: Netlify
date_fetched: 2026-07-08
date_published: 2026-07-08
---

AXIS is an open-source scoring framework Netlify built to quantify how well their platform serves AI agents — a concept they call Agent Experience (AX). Think Lighthouse, but for agent-facing services.

## How AXIS Works

Users provide a scenario — a JSON file containing a prompt and a scoring rubric — point it at an agent, and AXIS runs the agent against the endpoint. It supports 22 agents natively. Each run captures tool calls, responses, and recovery attempts, scoring across four dimensions: Goal achievement, Service, Environment, and Agent. The output is a 0–100 score and an HTML report.

## Netlify's Self-Test Results

Three scenarios tested with Claude Code and Codex, each run both cold ("no-context") and "with-skill":

| Scenario | Agent | Condition | Score | Time | Tokens |
|---|---|---|---|---|---|
| check-task-status | codex | no-context | 68 | 22.3s | 78,260 |
| check-task-status | codex | with-skill | 98 | 25.4s | 87,563 |
| check-task-status | claude-code | no-context | 78 | 45.1s | 131,163 |
| check-task-status | claude-code | with-skill | 92 | 27.6s | 85,773 |
| delegate-local-wip | codex | no-context | 63 | 66.1s | 226,082 |
| delegate-local-wip | codex | with-skill | 98 | 28.9s | 119,280 |
| second-opinion | claude-code | no-context | 69 | 139.9s | 450,558 |
| second-opinion | claude-code | with-skill | 95 | 109.0s | 240,509 |

Overall AXIS score for Netlify Agent Runners: 84/100.

Key findings:
- Skills boosted scores by an average of 26 points across all scenarios
- Every "with-skill" run reduced both time and token cost versus the no-context baseline
- The most dramatic improvement: delegate-local-wip with Codex jumped from 63 to 98, while time dropped from 66s to 29s and tokens nearly halved

## The Context Pipeline Vision

Netlify found that agent context currently lives in "four or five hand-maintained places" that tend to drift from the human-facing documentation. The team is working on a context pipeline that automatically derives, tests, and publishes agent context from repos that already hold docs. The vision: "New docs can't ship without a deliberate decision about their agent context." AXIS becomes the verification layer — running scenarios on every PR and treating score regressions like broken tests.

## Context

The article notes that interest in Agent Experience gained traction after a Queen's University study "found that 97% of MCP tool descriptions had quality issues" — not because the protocol itself was flawed, but because no one was designing with the agent as the end user.

Auth0's Group Product Manager Bharath Natarajan is quoted calling Agent Experience "a core part of how developers evaluate identity infrastructure" and noting that Auth0 is "proud to be a founding contributor to AXIS."

## Key Quotes

- "That's what happens when an agent has the context it needs rather than guessing."
- "New docs can't ship without a deliberate decision about their agent context."
- "Agent Experience shouldn't be something we audit once or periodically."
- "The belief has aged well. But belief isn't a standard."
- "A finding is a data point. A practice is what makes progress."
