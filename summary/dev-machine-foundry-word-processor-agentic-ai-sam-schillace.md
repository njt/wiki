---
url: https://commandline.microsoft.com/dev-machine-foundry-word-processor-agentic-ai-sam-schillace/
title: "How my dev machine built 1,111 word-processing features and forgot to add a ruler"
author: Sam Schillace
date_fetched: 2026-07-18
date_published: 2026-07-08
---

Sam Schillace (Deputy CTO at Microsoft) describes building a "dev machine" — an agentic loop that autonomously built a proof-of-concept Microsoft Word clone. Over 39 days it ran 565 sessions, made 3,706 commits, and produced roughly 350,000 lines of TypeScript.

The project survived three failed attempts before the breakthrough. The first two collapsed under complexity or broke when collaboration was added retroactively. The winning pattern emerged from an unrelated experiment where Schillace told an agent to "keep adding features" and left for the weekend — it ran 38 hours straight, spawning 440 sub-agents across 167 commits.

A "dev foundry" is the scaffolding that makes a dev machine work: progressive machine-readable specs, a clean event/task stack where the main loop only orchestrates, and a strong test environment the system can write against. The architecture spec was 947 lines written in about an hour; the loop compounded on it to produce 621 commits in the first week.

The central failure mode is **priority inversion**. The machine mapped 1,111 obscure OOXML properties — deterministic tasks that always passed — but never built a basic ruler, which sat in the candidate list for 22 sessions and 25 days with zero lines written. The machine chose the reliable green checkmark every time. "The machine is honest, but it isn't strategic. Strategy is still my job."

Key scar tissue: robustness cannot be arrived at incrementally — every new machine fails its first overnight run. A three-layer recovery stack (entrypoint retry, host watchdog, host monitor) is essential. Multiple machines entangle when shared patterns mix with per-project customization.

The highest-leverage hour is the spec, not the code. The loop compounds on what you give it — but it has no judgment, and it will faithfully deliver 1,111 features and no ruler unless you show up to say which one matters.
