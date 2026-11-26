---
url: https://claude.com/resources/articles/building-always-on-agents
title: "Building always-on agents"
author: William Hu (Anthropic)
date_fetched: 2026-11-26
date_published: 2026 (recent)
topics:
  - agent-architecture
  - agent-memory-and-context
---

Anthropic's engineering guide to always-on agents — agents that run indefinitely across many sessions, waking on events and acting unattended. The core claim: the agent loop is unchanged; what's new is the *surrounding architecture*, built from five primitives — event inbox, timers, memory, permissions, and outbox.

The resurgence is credited to better model judgment and cheaper intelligence (Opus 5.5 costs 78% less per solved task than Opus 5 on long-horizon benchmarks). An always-on agent is "not actually always on": it sleeps between **runs**, each run operating inside a **session** (the message/tool transcript) and persisting state for the next run.

- **Event inbox**: every trigger (webhook, schedule, background job) writes to a durable table; a single worker per inbox picks up pending events and starts a run. Ack fast, treat the inbox as source of truth, batch related events, one active worker at a time.
- **Timers**: follow-ups (agent-set deadlines) and schedules (recurring tasks) in their own store, fired by a polling scheduler that injects events into the inbox.
- **Memory**: a directory-of-files store on durable storage — user instructions, user notes, past messages (to avoid repeats), and per-task progress files — with only an index loaded at run start. Rule of thumb: store only what the agent can't look up again; save pointers otherwise. Optional "dreaming" pass curates memory across sessions.
- **Permissions**: the riskiest part, since runs are unattended. Credential injection via an out-of-environment proxy, classifier-based auto mode (from Claude Code), and deferred tool calls that pause a session mid-call and resume when the user's answer arrives as an inbox event.
- **Outbox**: a durable outgoing-message table plus dispatcher, needed only when you want pacing or error recovery; the run ends by sending, staying silent, or deferring. Tune message volume with evals — "start quiet and earn more of the user's attention over time."

The pieces compose into a cycle: events start runs, runs set timers and update memory, outbox delivers results, user replies come back as events — brief runs, indefinite loop.
