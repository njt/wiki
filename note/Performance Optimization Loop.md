# Performance Optimization Loop

Steve Gordon's structured, repeatable methodology for turning performance work from reactive firefighting into a proactive engineering discipline: monitor in production, profile hotspots, benchmark a baseline, apply small targeted changes, re-benchmark, document, deploy, and validate. The loop is applied to a real .NET OpenTelemetry SQL sanitizer, yielding 86% allocation reduction and 46% CPU reduction across seven incremental stages.

---

## Key Quotes

> "Premature optimization is the root of all evil. … Yet we should not pass up our opportunities in that critical 3%."

Gordon restores the full context of the most-misused quote in performance engineering. The abbreviated version is deployed to shut down performance conversations; the full version accepts that a small number of situations genuinely warrant optimization. The difference between the two is data. This is the best single-sentence defense of measurement-driven optimization.

> "Performance, unlike functional bugs, is quite insidious. It doesn't necessarily rear its head immediately. It will degrade over time until a point where it hits you."

The degradation argument is the practical reason to treat performance as a discipline rather than a crisis response. Functional bugs have cause-and-effect that's easy to correlate — a null reference exception crashes immediately. Performance degradation compounds across releases: 200ms → 250ms → 300ms → 400ms, each change too small to notice individually, catastrophic in aggregate. This is why monitoring and SLOs must be in place *before* deployment, not after complaints.

> "Never trust a user blindly. … Go back to your production data and ask it that kind of question."

User reports are triggers for investigation, not conclusions. The same user who says "it's slow" might be on a particular feature plan that sees different latency, or comparing against yesterday when they briefly had better conditions. Production traces and metrics are the only tool that can distinguish between "this user is experiencing degradation," "this customer tier is experiencing degradation," and "this user is on a different network."

> "It was fast locally" is a bias you must avoid.

The compendium of reasons: local dev machines don't match production GC modes, cache configurations, database latencies, concurrency patterns, or network topologies. Local benchmarks compare like-for-like on the dev machine — they're essential for the inner optimization loop — but they tell you nothing about whether the gain translates to real users. Only production data answers that.

> "The goal is not the fastest code possible. It's code that's fast enough, but easy to maintain and delivering good results for you."

The stopping rule the field lacks. Gordon offers several: diminishing returns in benchmarks, production metrics hitting their targets, the user ticket closing, cost reduction goals met. But the more honest admission is "it's quite addictive. Watching those numbers go down is fun." The discipline is not technical; it's knowing when to walk away.

> "I tend to see allocations as I'm reading code, which is both good and bad. I mean, it's slightly better than seeing dead people."

The best one-liner in the talk, and also the most dangerous. Developer intuition about performance is the gateway to premature optimization. Gordon's own advice — go back to production data before acting on anything you "see" in the code — is the antidote to the skill he describes developing.

## Key Themes

- **#pattern** The Performance Optimization Loop: monitoring → profiling → tests → baseline benchmarks → small targeted changes → re-benchmark → document → repeat. Inner loop iterates on methods; outer loop validates against production. The structured process is the contribution — most practitioners do fragments of this intuitively but not systematically.

- **#tool** The .NET optimization stack: JetBrains dotTrace/dotMemory with Profiler API (NuGet package for in-code profiling control), Benchmark.NET (the de-facto .NET benchmarking library), `Span<T>`/`Memory<T>` (zero-allocation memory views), `SearchValues<T>` (ultra-fast sequence search), `ref struct` (guaranteed stack allocation), `ArrayPool<T>` (rented buffers), `ConcurrentDictionary` with approximate count.

- **#concept** Performance as a three-dimension impact triangle: user experience (latency, smoothness), cost (CPU/memory → infrastructure spend), and reliability (GC pauses, throttling, cascading failures). Improving one dimension helps the others, but not equally — the priority depends on business context. This is the framework for answering "how much should we invest?"

- **#concept** Production data as the ultimate trigger and validator. Local benchmarks, dev-machine tests, and developer intuition are all inputs — but production monitoring data (APM, traces, SLOs) is the only thing that can tell you whether performance work actually mattered. This is the strongest through-line in the talk and the claim least likely to be disputed.

- **#pattern** Small targeted changes over batch optimization. If you see six opportunities within a method, apply them one at a time. Change one thing, re-benchmark, re-test, document. Batch changes can hide regressions — one of your six "improvements" might make things worse, but you won't know which. The scientific approach costs wall-clock time but buys confidence.

- **#concept** AI-assisted optimization as a filter-and-verify pipeline. Gordon asked Copilot for 10 optimizations: two were insane, one hallucinated an API, seven were plausible. He manually tested each, kept the ones that improved benchmarks without breaking tests, discarded the rest. The human is the gate, not the source of ideas.

## Critical Analysis

**The talk's strongest contribution is the loop itself, not the .NET specifics.** The seven optimization stages (pre-sized StringBuilder → static reuse → ConcurrentDictionary → Span slicing → rented arrays + ref struct → SearchValues + algorithmic shortcuts → return-original-string) are a worked example that demonstrates the methodology. The methodology transfers to any runtime; the specific optimizations don't. The talk would be stronger if it acknowledged this separation explicitly — as it is, a JVM or Go developer watching might miss that the loop, not the span tricks, is the reusable artifact.

**The "3%" framing is doing a lot of work.** Gordon characterizes this as a rare scenario — trading platforms, health systems, observability libraries. But the definition is slippery: the talk's own example (an OpenTelemetry instrumentation library) is in the 3% because observability tools must not affect application performance. By that logic, any library consumed by other teams' hot paths is in the 3%, which is a much larger category than "trading platforms." The "most of you probably aren't working on those kinds of applications" line undersells how many engineers work on shared infrastructure that's performance-sensitive.

**The stopping problem is acknowledged but not solved.** Gordon offers several heuristics — diminishing returns, production SLOs met, user ticket closed — but admits none are quantitative. The talk doesn't provide the thing it most needs: a decision framework for when to stop optimizing. The closest it gets is "46% CPU and 86% allocations was enough," which is a result, not a rule. Every practitioner doing this work will face the same question and get the same non-answer.

**The AI section is the most forward-looking and the least developed.** At 2–3 minutes in a 60-minute talk, it gets token coverage. But the pattern — AI proposes, human filters, benchmarks verify — is a template for how AI-assisted optimization should work across any domain. The talk predates the era where coding agents can do this autonomously; a 2026 update would need to address the verification problem when the agent proposes AND implements the optimization. [[Eval-Driven Development (Airbnb)]] offers an evaluation framework that could plug directly into this loop.

**The documentation practice is undervalued in the presentation but is arguably the most transferable advice.** Gordon comments the optimized code and warns future maintainers to re-run benchmarks before modifying it. This is the difference between a one-time optimization and a maintained optimization — without it, the next developer who touches the method will "clean up" the spans back into substrings and regress everything. The talk mentions this in one slide; it deserves a section.

**The relationship to [[Span-First CSharp — Designing Around SpanT]] is complementary but distinct.** Where the span-first guide is a *design pattern* for building allocation-free APIs, Gordon's talk is a *methodology* for finding and fixing allocation hotspots. The span-first guide tells you how to write a span-based API; Gordon tells you how to *decide* whether and where to do it. The two pages should be read together: one is the tool, the other is the process for choosing when to reach for the tool.

**The talk intersects with [[Lorenz and Little — How Much Does Your Tail Cost]] at the measurement layer.** Brooker gives you the analytical tool (which part of the latency distribution drives cost?); Gordon gives you the process for fixing what you find. The two together form a complete performance engineering workflow: diagnose with Lorenz curves, optimize with the loop, validate with production data. Neither page makes this connection explicit — but a practitioner who reads both will.

**Gordon's optimization-stages narrative is a masterclass in incrementalism.** Stage 1 (pre-sized StringBuilder) → Stage 2 (static reuse with atomic concurrency control) → Stage 3 (ConcurrentDictionary + approximate count) → Stage 4 (Span slicing for Substring elimination) → Stage 5 (remove StringBuilder entirely, switch to rented char[] + ref struct) → Stage 6 (return original string when sanitization is a no-op). Each stage is independently testable, benchmarkable, and reversible. This is the scientific method applied to performance, and it's the thing most missing from the "just make it faster" instinct.

**The unanswered questions list (from the gist's key-points section) is unusually thorough for a conference talk summary** and highlights the structural gaps: SLO threshold-setting, engineering-cost vs. infrastructure-savings calculus, scaling to distributed systems, cultural/organizational barriers, quantitative definitions of "diminishing returns," and systematic detection of hidden allocations (the boxed struct that dotMemory missed). A talk that raises these questions and doesn't answer them is more honest than one that pretends it's all solved — but the reader who finishes wanting frameworks for these questions will need to look elsewhere.

## Related Pages

- [[Software Engineering Craft]] — Hub page; performance engineering is a craft discipline, not a tool-specific skill
- [[Span-First CSharp — Designing Around SpanT]] — The design pattern for the span-based optimizations Gordon applies in stages 4–6; read this to learn the *how*, Gordon to learn the *when*
- [[Lorenz and Little — How Much Does Your Tail Cost]] — The diagnostic tool for the measurement step that triggers the loop; Brooker tells you *where* to optimize, Gordon tells you *how*
- [[Unnecessary Optimization in Rust]] — The compiler-trust thesis applied to a different language; where Schwartz argues "write code the optimizer can reason about," Gordon shows the process for discovering what to optimize in the first place
- [[Eval-Driven Development (Airbnb)]] — The eval framework that could be applied to AI-assisted optimization suggestions: benchmark results as evals, regression detection as the quality gate
- [[Queues Don't Fix Overload]] — Hebert's argument that queues treat symptoms, not causes; Gordon's loop is the causal approach to performance problems
- [[Lean, Not Backpressure]] — Lean manufacturing applied to software; Gordon's optimization loop is single-piece flow applied to code changes
- [[React useMemo and useCallback]] — Josh Comeau applies the same measurement-first discipline to React: profile with the React Profiler before reaching for `useMemo` or `useCallback`, and wrap only the hot paths — the frontend instance of Gordon's loop
- [[Push Ifs Up And Fors Down]] — Alex Kladov's "push fors down" heuristic (batch as base case, scalar as special case) is the code-organization counterpart to Gordon's measurement-driven loop; batching amortizes setup costs and unlocks vectorization, but only Gordon's loop tells you whether it actually moved the needle in production
- [[The Wicked Reason Removing Code Beats Better Scheduling]] — Alex Russell scopes Gordon's loop precisely: it's the *decent→excellent* technical challenge. Most teams, Russell argues, are stuck on *poor→decent*, which is a management/culture problem that code removal — not measurement-driven optimization — must solve first

---
*Sources: [[raw/application-performance-optimisation-in-practice]], [[summary/application-performance-optimisation-in-practice]]*
*Last updated: 2026-08-07*
