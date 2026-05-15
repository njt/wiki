---
title: "Memory Is a Mistake"
url: https://xcancel.com/manthanguptaa/status/2015780646770323543
author: Manthan Gupta (@manthanguptaa)
date_fetched: 2026-05-15
date_published: 2026-01
section: "Memory & Context"
---

# Memory Is a Mistake — Manthan Gupta

Full analysis at: https://manthanguptaa.in/posts/memory_is_a_mistake/

Original tweet thread breaking down Clawdbot/OpenClaw memory architecture: https://xcancel.com/manthanguptaa/status/2015780646770323543

## Tweet Context

Manthan Gupta broke down the Clawdbot/Moltbot/OpenClaw memory architecture in a viral tweet thread. His key finding: the bot heavily relies on tools to reference memory, but models aren't trained to use those tools consistently. This observation led to a comprehensive blog post comparing memory architectures across ChatGPT, Claude, OpenClaw, and Hermes.

## The Core Thesis

"Most AI products do not need better memory. They need better product design."

The storage side is not the hard part. The critical, overlooked component is the retrieval policy — the heuristic deciding which remembered thing gets pulled into which future prompt.

## Four-System Comparison

1. **ChatGPT** — injected profile approach. Always injects memory block. Simplest, most brittle.
2. **Claude** — on-demand retrieval. Model decides when to search. Tools: conversation_search, recent_chats.
3. **OpenClaw** — Markdown workspace with hybrid search. Agent issues semantic + keyword search.
4. **Hermes** — hot/cold split with explicit tiers. Best design per the author.

## Hermes Design Principles (the "cure")

- Separate hot memory from cold recall — always-injected tier capped strictly
- Prompt stability as first-class constraint — memory frozen at session start
- Memory is plural — facts, episodes, skills are distinct retrieval problems

## Six Failure Modes

1. Output quality degradation — memory bleeds into unrelated contexts
2. Debugging difficulty — two pipelines (request + memory) with only one logged
3. Context rot — accuracy drops well before the window fills (as early as 32k tokens)
4. Privacy issues — CIMemories: right domain, wrong granularity
5. Attack surface — persistent prompt injection via memory (Unit 42 PoC)
6. Personality drift — 97% sycophancy rate in long-term-memory systems (PersistBench)

## Pre-Shipment Checklist

Five questions teams must answer before shipping memory. If mostly no, ship visible settings, scoped project state, and explicit task briefs instead — citing Cursor's .cursorrules, Claude Projects, Zed's .rules, ChatGPT Custom Instructions, and Linear task context as examples that work because they are "legible, editable, and scoped."
