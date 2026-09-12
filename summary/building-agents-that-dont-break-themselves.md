---
url: https://fly.io/blog/building-agents-that-dont-break-themselves/
title: "Building Agents that Don't Break Themselves"
author: Daniel Botha
date_fetched: 2026-07-08
date_published: 2026-06-08
topics:
  - security-and-sandboxing
  - agent-architecture
---

Daniel Botha argues that AI agents sabotage themselves when they run
destructive commands in their own environment, and that the fix is
architectural: separate the agent's reasoning loop (the "brain") from its
shell execution (the "hands"). The agent process lives on a durable machine;
commands run in an ephemeral sandbox — a "padded room" you'd be happy to set
on fire.

Two case studies illustrate the spectrum. SpriteDoc spins up one throwaway
sandbox per user session, injecting auth tokens per-command so no secret
rests on shared disk. Hermes Agent keeps one sandbox per task and resumes it
across sessions, trading ephemerality for persistence. Both skip confirmation
prompts because the sandbox itself is the security boundary.

The article doubles down on defense in depth: even an agent already inside a
sandbox should dispatch commands to a *separate* sandbox. Checkpointing
before risky steps — copy-on-write, fast enough to be reflexive — means that
after an agent nukes `/usr/bin/python3` and `/usr/bin/git`, everything is
back in about nine seconds.

The core aphorism: *"Telling your agent to be careful is silly. Just make it
do things somewhere it doesn't have to be."*
