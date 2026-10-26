---
url: https://www.dbreunig.com/2026/10/24/the-state-of-context-engineering.html
title: "The State of Context Engineering"
author: Drew Breunig
date_fetched: 2026-10-26
date_published: 2026-10-24
topics:
  - agent-memory-and-context
  - agent-architecture
---

A lightly edited transcript of Drew Breunig's AGNTCon talk "Context Fails & Fixes", a state-of-the-field survey of context engineering from late 2024 to 2026, ahead of his O'Reilly book *The Context Engineering Handbook*.

The talk opens with the "RAG is dead" moment: the bet that million-token windows (Gemini 1.5, Llama 4 Scout's 10M) would let you "just put everything in the context." Breunig catalogues the four failure modes that killed that hope — **context poisoning** (bad facts the model can't get past, illustrated by Gemini 2.5 playing *Pokémon FireRed* while seeking lemonade-era tea), **context distraction** (long contexts drown base knowledge; the "Do as I Say, Not as I Do" paper shows even Opus 4.6 follows contradictory in-history examples over instructions), **context confusion**, and **context clash**. Long contexts, he concludes, "let you fit in more errors."

The response was six harness tools: context augmentation, tool loadout (he recounts an enterprise team loading 600 skills at startup), quarantine (subagents), pruning, offloading, and summarization/compaction. He demos them in ~16 lines of DSPy, using a tiny decision model ("Jev") to do loadout, pruning judgment, and pre-compaction relevance filtering. Naming has shifted — prompt engineering (2023) → context engineering (2025) → harness engineering — but "the job is the same": choosing when to be deterministic in code and when to be probabilistic in the model.

For 2026, the pitch is models that drive their own harnesses over long horizons (GPT-5 ~3.4h on METR, Gemini 3.1 6.4h, Opus 4.6 ~12h), which creates two new failure modes: **fighting the weights** (models overfit to their training environments; Claude Code's system prompt begs for parallel tool calls seven times because training was serial) and **context inertia** (models hyper-focus on one hypothesis past the point of usefulness — documented independently by Anthropic's Memento harness, Microsoft's Fork-and-Flush paper, and UC Berkeley's *Scaling Discovery Requires Forgetting*). The fix is startlingly old: periodic context clearing with artifacts preserved in the environment. His practical advice: around 200K tokens, just delete your context.

His prediction for the next frontier: personal agents (Hermes, OpenClaw, etc.) running for six-plus months accumulate skills and memories that conflict and rot — "strategic forgetting" as the context problem of tomorrow.
