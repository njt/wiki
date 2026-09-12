---
url: https://www.oreilly.com/radar/building-organizational-intelligence/
title: "Building Organizational Intelligence"
author: "An O'Reilly engineering leader (unnamed)"
date_fetched: 2026-08-07
site: "O'Reilly Radar"
topics:
  - agent-coding-workflow
---

An O'Reilly engineering leader describes how they built an organizational intelligence system that combines internal data, AI reasoning, expert frameworks, and human review to produce actionable, defensible recommendations rather than generic data summaries.

The article opens with a concrete anecdote: an AI agent analyzed a team's workload and produced a thorough data dump — headcount, velocity, backlog — that was "easy to agree with and difficult to act on." Adding the O'Reilly Expert MCP server transformed the output into a framework-grounded recommendation citing Google SRE's toil threshold, diagnosing a structural problem rather than a staffing shortage.

The author traces how engineering has moved from waterfall (central plans, high visibility) through agile (autonomous teams, splintered information) into the agentic era (extraordinary output, near-zero visibility). The four-step recipe:

1. **Map your information hierarchy** — strategic intent down to operational detail, making explicit which sources carry authority and how decisions get made.
2. **Connect systems via MCP and write a skill file** — MCP gives access to data; the skill file (CLAUDE.md) tells the model how to reason with it. Without the skill, you get retrieval; with it, you get analysis.
3. **Add an expert review layer** — the O'Reilly Expert MCP grounds analysis in named frameworks (Google SRE, Team Topologies, *Accelerate*, Wardley mapping) with traceable citations. This is the step that "changes everything," shifting output from generic advice to defensible recommendations a director can interrogate.
4. **Build human-in-the-loop review** — AI can't know what isn't documented. Human judgment on context, priorities, and risk tolerance is nonnegotiable.

The author also describes **Superanswers**, an internal system using GitHub as the source of truth for AI-generated research documents and their discussions, turning the AI from a one-shot report generator into a participant in ongoing organizational conversation.

The organizing principle: *Organizational data provides local evidence about what is happening in your specific context. Expert frameworks provide accumulated practitioner knowledge about how to think about problems of that kind. Good organizational judgment requires both.*
