---
url: https://blog.bl00cyb.org/2026/07/agents-and-acquiring-debt/
title: Agents and Acquiring Debt
author: bl00cyb (with Tom Henderson and Mark)
date_fetched: 2026-08-25
date_published: 2026-07
topics:
  - software-engineering-craft
  - agent-coding-workflow
---

# Agents and Acquiring Debt

A blog essay — cowritten with Tom, fact-checked by Mark — that reframes technical debt for the age of coding agents. The core claim: AI doesn't eliminate debt, it shifts *when* debt is acquired. Because gathering information early is now cheaper, teams can defer hard-to-reverse decisions to "the last responsible moment," which changes the *kind* of debt they take on rather than the amount.

The authors separate two debt shapes. Traditional technical debt — Cunningham's "expedient-with-knowledge-of-cost" — was at least decided knowingly. Agent-generated debt is accepted-without-agreement: not even surfaced for awareness, and produced at far higher volume. Out of this they name **comprehension debt**, the AI-native type and "the new highest-interest loan you can take out against your code." It accrues when you LGTM an agent's edit without understanding it. Agents answer *what*, not *why*: they hold current state and commits, but not the folklore or the rejected alternatives with reasoning. "Every agent is the new hire on day one, forever."

The remedies: record decisions when they're made — ADRs, which agents can now write at decision time and are their hungriest readers. Cheap refactors and purpose-built linters are newly affordable, but paydown itself churns foundations and generates its own comprehension debt. And deferral has holding costs: don't let "maximizing information" become procrastination. Mark's metaphor: "AI companies that sell coding agents are the equivalent of predatory credit card companies offering introductory cards on college campuses to freshmen."

*Sources: [[raw/agents-and-acquiring-debt]]*
