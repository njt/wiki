---
url: https://www.seangoedecke.com/you-should-all-be-asking-way-more-questions/
title: "You should all be asking way more questions"
author: Sean Goedecke
date_fetched: 2026-10-03
date_published: 2026-09 (approx; not stated verbatim)
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

Sean Goedecke argues that interrupting explanations with short confirmation questions — roughly one every thirty seconds — is the correct way to actually understand a plan, and that the habit matters more now that many of our "colleagues" are AI agents.

His core mechanism is the early misunderstanding that balloons: a small misread in the first minute compounds into everything built on top of it, which is why batched questions at the end don't work. He notices most people don't ask questions because they have no skin in the game — they trust the senior engineer to know what's going on — but once you're the one who must act on the plan, that trust evaporates and the questions come naturally. The discipline behind the questions is "building the plan in your head": visualising actual data flow, service-to-service auth, and what gets persisted where, which is how a vague phrase like "service X stores data" (when X only touches an ephemeral Redis) surfaces a missing dependency before any code is written. He recalls an event-driven system so elegant it made data-residency impossible and had to be abandoned — the kind of unworkable feature early questions catch.

The turn of the piece is that AI agents are "inherently unreliable" colleagues: models rarely make code mistakes but make design mistakes constantly — assuming two services can talk, forgetting on-prem constraints — because "language models are always on their first day" (no continuous learning). So pepper them with questions: does this exist elsewhere in the codebase, does service Y really support this auth, is this subsystem actually needed for requirement Z, why are we touching this file? About half the time he concludes the model erred. He'll stop asking when that stops happening — and jokes that his company may then want him in an architect role. Closing claim: nobody understands complex software products; with domain knowledge you will routinely correct both powerful models and principal engineers.
