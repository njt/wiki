---
url: https://matklad.github.io/2023/11/15/push-ifs-up-and-fors-down.html
title: Push Ifs Up And Fors Down
author: Alex Kladov (matklad)
date_published: 2023-11-15
source_site: matklad.github.io
---

A short blog post articulating two related rules of thumb for code structure from Alex Kladov (matklad), the creator of rust-analyzer and a TigerBeetle engineer.

**Push Ifs Up:** When an `if` condition appears inside a function, consider moving it to the caller instead. This often arises with preconditions — rather than checking a precondition internally and silently doing nothing, enforce it at the call site via types or asserts. The pattern can become viral, resulting in fewer checks overall. The deeper motivation is that control flow and conditionals are sources of bugs; pushing `if`s up centralizes branching logic in a single function where redundancies and dead conditions become visible. A related technique is the "dissolving enum" refactor, where duplicate branching on the same enum variant across multiple locations collapses into a single dispatch.

**Push Fors Down:** From the data-oriented school: few things are few, many things are many. Introduce a concept of a "batch" of objects and make batch operations the base case, with scalar versions as a special case. The primary benefit is performance — batching amortizes setup costs, enables flexible processing order, and unlocks vectorization and struct-of-array tricks. The most striking example is FFT-based polynomial multiplication, where evaluating a polynomial at many points simultaneously is faster than individual evaluations.

The two rules compose: pulling a condition out of a hot loop avoids repeatedly re-evaluating it, removes a branch, and potentially unlocks vectorization. Kladov notes this pattern works at both micro and macro levels — it's the architecture of TigerBeetle, where the data plane operates on batches of objects to amortize control-plane decision costs.
