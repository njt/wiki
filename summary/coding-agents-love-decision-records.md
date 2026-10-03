---
url: https://www.oreilly.com/radar/coding-agents-love-decision-records/
title: "Coding Agents Love Decision Records"
author: Duncan Davidson
date_fetched: 2026-10-03
date_published: unknown
topics:
  - agent-coding-workflow
  - agent-memory-and-context
---

Duncan Davidson (republished on O'Reilly Radar) argues that Architectural Decision Records (ADRs) give coding agents durable project context: agents arrive with little memory and a narrow view of the codebase, and explicit recorded intent spares them from code archaeology and stops them mistaking implementation details for foundational rules.

But ADRs interact with agents in two failure modes. First, agents adhere to accepted decisions *more* rigidly than humans — Davidson describes an agent that preserved an outdated storage abstraction across a new feature because an ADR still called it mandatory, adding a compatibility layer rather than flagging the mismatch. Second, if you invite agents to update decisions, they preserve the deliberation: every clarification becomes an amendment, implementation details become rules, and the prose becomes overlitigated and unreadable for humans.

His remedy is a set of explicit conventions in the project's AGENTS.md: accepted ADRs are binding, proposed ones are context, superseded ones are history; keep each ADR succinct, state each rule exactly once and cross-reference, let Git history serve as the changelog instead of amendment logs. The closing principle: "An agent doesn't need the transcript of every argument. It needs the ruling that governs today and clear permission to stop when the ruling no longer fits."
