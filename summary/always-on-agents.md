---
url: https://rubriclabs.com/blog/always-on-agents
title: "Always-On Agents"
author: Rubric Labs
date_fetched: 2026-11-09
date_published: 2026 (recent)
topics:
  - agent-architecture
  - personal-agents
---

Rubric Labs' design essay on why the chatbot/agent form factor doesn't match the "Jarvis" mental model, and what architecture an always-on agent needs. They argue the disagreement about whether we've reached AGI is "more about form factor than it is about model capabilities" — the gap is between ephemeral, reactive, context-dependent sessions and a single-threaded, proactive, situationally aware agent.

The architectural prescriptions are simple and concrete: make the agent single-threaded, but give it tools to manage its own context (`foldContext`/`dropContext` instead of third-party compaction, which "lobotomizes the agent's momentum") plus sub-agents it dispatches heavy work to. For proactivity, give the agent a **heartbeat** — a message pushed into the chat history that wakes it on a cadence or on its own schedule — and make **messaging a tool**, so reaching the user is a decision rather than the automatic end of a turn.

Situational awareness comes from broad access (email, browser, file system — "one simple tool" of computer use, mirroring how terminal access made coding agents great) plus a notifications inbox with webhooks and polling jobs pushing events in, filtered by a smaller classifier model so unimportant messages pile up instead of waking the agent every minute.

The hard part, and their real focus, is **discernment**: models trained on task-completion RL are great when the signal to start is clear but struggle in ambiguous environments — acting on Slack messages not addressed to them (then producing "useless and embarrassing" design docs) or missing real triggers. Their prescription: start the agent shy, give it a clear mandate ("doing nothing is totally fine"), let it ask rather than guess, and give it memory of how the user reacted when it acted. They call for new RL environments that reward discernment and memory use.
