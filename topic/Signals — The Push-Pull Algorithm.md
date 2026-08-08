# Signals — The Push-Pull Algorithm

Willy Brauner builds a signals implementation from scratch in TypeScript, walking through the two halves of the algorithm — push-based eager invalidation and pull-based lazy re-evaluation — that power fine-grained reactivity in Solid, Vue, Preact, Angular, and Svelte. The result is a dense, working reference implementation in ~80 lines that demystifies the global-stack dependency tracking trick.

---

## Key Quotes

> A Signal is an abstraction that represents a reactive value that can be read and modified.

The elevator pitch. Signals aren't just event emitters — they're the nodes in a reactive graph. Reading a signal during a computed's execution creates an edge in that graph automatically.

> A notification is immediately *pushed* to its subscribers when the signal is updated.

But critically, Brauner notes that signals using the push-pull algorithm don't push *values* — they push *invalidation*. The subscriber learns "your cache is stale," not "here's the new value." That distinction is the entire game.

> Computeds are lazy — they are invalidated (not updated) whenever one of their dependencies changes, and re-evaluated only when accessed.

This is where signals beat the naive pub/sub approach. Without laziness, a single `count.value = 5` at the root of a deep dependency tree triggers O(depth) recomputations, many of which produce intermediate values nobody ever reads. With pull, you only pay for what you consume.

> A computed being read has no knowledge of the entire tree; it only knows its sources (dependencies) and subscribers (dependents).

The graph is local. Each node knows its immediate neighbors. The global reactive behavior emerges from these local contracts. This is why signals scale — no central coordinator, no topological sort of the full DAG.

> What makes signals interesting is not just that they update some UI, but *how* they propagate change through a reactive graph.

Brauner's thesis: the algorithm is the interesting part. The UI update is just one consumer among many.

---

## The Algorithm, Step by Step

### 1. Push (Signal → Subscribers)

```
count.value = 5
  → subscribers.forEach(setDirty)
  → doubleCount.dirty = true
  → doubleCount.subscribers.forEach(setDirty)
  → plusOne.dirty = true
```

Invalidation propagates synchronously down the graph. Nothing recomputes yet — just flags flipping.

### 2. Pull (Read → Recompute)

```
doubleCount.value
  → dirty? yes → push ComputeContext to STACK
  → fn() runs, reads count.value
  → count's getter sees STACK has context
  → registers setDirty as subscriber, addSource for cleanup
  → fn() returns 10
  → dirty = false, pop STACK
  → return 10
```

The STACK trick is the heart of auto-tracking. The currently-executing computed announces itself, and every signal read during execution registers the computed as a dependent. No manual dependency arrays, no `useMemo` dependency lists — the dependencies are whatever you actually read.

### 3. Dynamic Dependency Graphs

The `addSource` / cleanup pattern means dependencies can change between evaluations:

```typescript
const dynamic = computed(() => {
  if (useExpensive.value) return expensive.value * 2
  return cheap.value * 2
})
```

When `useExpensive` flips, the old subscription to `expensive` is cleaned up, and the new one to `cheap` is established. The graph rewires itself.

---

## Key Themes

- **#concept** — Reactive programming as spreadsheet semantics: declare relationships, let the system maintain consistency
- **#pattern** — Push-pull as two-phase cache coherence: invalidation propagates eagerly, recomputation happens lazily
- **#concept** — Dependency auto-tracking via global execution context (the STACK trick) — the "magic" that makes signals ergonomic
- **#tool** — TC39 proposal-signals (Stage 1) pushing toward native JavaScript standardization
- **#pattern** — Dynamic dependency graphs: subscriptions are rebuilt on every evaluation, not declared once

---

## Critical Analysis

**The STACK trick is brilliant — and hostile to debugging.** A global mutable array that code reads and writes as a side channel is exactly the kind of implicit state that makes functional programmers twitch. It works because JavaScript is single-threaded and synchronous — computed evaluations can't interleave. But throw a `setTimeout` or an async read inside a computed, and the model silently breaks in ways that will have you staring at stack traces for hours. The article mentions this only in passing; it's a bigger constraint than it looks.

**This is cache coherence, not a new idea.** The computer architecture people solved this decades ago: write-invalidate protocols for shared caches. Signals are just that pattern applied to application state. A signal is a cache line; a computed is a cached computation; `setDirty` is an invalidation message. Brauner alludes to this lineage ("push-pull functional reactive programming" by Conal Elliott) but the direct analogy to CPU cache protocols would make the concept more accessible to systems programmers.

**The article's strength is its concreteness.** Rather than hand-waving about "reactivity," Brauner writes the damn code. The ~80-line TypeScript implementation is the real content — everything else is annotation. This is how technical writing should work: build the thing, then explain it. The interactive diagrams (tree visualizations showing invalidation propagating and recomputation pulling) complement the code well.

**What's missing: batching and transactions.** The implementation notifies on every `set`. Change three signals in a row, and you get three rounds of invalidation propagation before anyone reads anything. Real signal libraries batch updates — either explicitly (`batch(() => { a.value=1; b.value=2 })`) or by deferring effects to a microtask. This isn't a flaw in the article (it's explicitly a minimal implementation) but it's the first thing you'd hit trying to use this in production.

**The TC39 proposal is both exciting and concerning.** Native signals would eliminate the framework wars over which signal library to use — they'd be a platform primitive, like Promises. But the proposal is Stage 1, and platform standardization of UI primitives has a mixed track record (Web Components took a decade to become usable). The risk is that TC39 standardizes the wrong abstraction and we're stuck with it for 20 years.

**Why now?** Signals are 50 years old (reactive programming, 1970s) and 15 years old in JavaScript (Knockout.js, 2010). Their sudden dominance — adopted by every major frontend framework in the last 3 years — isn't because the algorithm improved. It's because React's reconciliation model hit its scaling limits, and signals provide genuinely fine-grained updates without a virtual DOM. The algorithm was always good; the context changed.

---

## Related Pages

- [[Phoenix LiveView]] — Server-side stateful processes pushing UI diffs; the same reactive model transposed to the server
- [[Gova]] — Declarative reactive GUI framework for Go: SwiftUI-inspired, call-site state identity
- [[State System]] — Organizational state layer with evidence-first commits and deterministic replay — signals at the org scale
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines; the end-to-end principle applied
- [[Apache Burr]] — State machines as the explicit version of what signals do implicitly
- [[Event-Driven vs Polling Architectures]] — The push side of signals is event-driven invalidation; the pull side is polling on read
- [[The Valley of Webhooks]] — The same push-to-pull inversion at the API integration scale: flip from provider-push webhooks to consumer-pull change logs, and the dedup/ordering/bootstrap stack collapses the way eager recomputation collapses under lazy signals
- [[React useMemo and useCallback]] — Josh Comeau's explainer makes the tradeoff concrete: React's `useMemo`/`useCallback` are the manual-dependency-array version of what signals do automatically. Signals eliminate stale-dependency bugs at the cost of a different primitive; React's explicit dependencies give more control at the cost of more discipline

- [[React Component Purity]] — React's complementary approach to the same reactivity problem: instead of tracking mutation through a graph, eliminate the tracking problem entirely by making renders pure calculations over immutable inputs. Signals surgically update; React re-renders the world from clean state. Two different bets on where the complexity should live

---
*Sources: [[summary/signal-the-push-pull-based-algorithm]]*
*Last updated: 2026-08-08*
