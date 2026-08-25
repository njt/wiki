---
url: https://infrequently.org/2026/08/notes-on-performance-remediation-strategies/
title: The Wicked Reason Removing Code Beats Better Scheduling
author: Alex Russell
date: 2026-08
date_fetched: 2026-08-25
---

Alex Russell responds to colleague Marko Ilić's post on scheduling work on the critical path, and surfaces the office discussion it kicked off. His thesis: when a team sets out to fix poor web performance, removing code should almost always be prioritized over reordering when that code loads.

The core argument is that scheduling is not the cheap shortcut it appears to be. Both code removal and resource reordering share the same largest cost — the investment required to deeply understand page behaviour — so reordering cannot be assumed cheaper or easier. Worse, deferred bytes still land on the main thread: JavaScript fetched late generates heavy "thuds" when residual allocations from background compilation arrive, stalls that show up in INP data but are maddeningly hard to trace. Systems that lean on scheduling to guarantee performance are brittle, coordination-heavy, and prone to use-case overfitting as every feature owner lobbies to load their code first.

Removing code, by contrast, is more durable, easier to defend, easier to reason about, and more resilient to changing conditions — including shifts in the user population as a product grows. Russell's closing framing: moving from poor to decent performance is a management and culture problem, while going from decent to excellent is a technical one. His general-purpose advice — reduce code sent to the client, move work to the server, and only reorder when those hit diminishing returns — positions scheduling as "special occasion food" reserved for teams already running strict size budgets and regression prevention.
