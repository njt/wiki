---
url: https://willybrauner.com/journal/signal-the-push-pull-based-algorithm
title: "Signals, the push-pull based algorithm"
author: Willy Brauner
date_fetched: 2026-06-21
date_published: 2026-03-23
topics:
  - software-engineering-craft
---

# Signals, the push-pull based algorithm

Author: Willy Brauner
Published: 2026-03-23
Source: https://willybrauner.com/journal/signal-the-push-pull-based-algorithm

## Summary

Brauner builds a signals implementation from scratch in TypeScript, walking through the push (eager invalidation) and pull (lazy re-evaluation) halves of the algorithm. Starting from a pub/sub signal primitive, he layers in computeds with automatic dependency tracking via a global STACK-based context, then combines both into the full push-pull pattern used by Solid, Vue, Preact, Angular, and Svelte. The article includes interactive diagrams and a full reference implementation.

## Key Sections

### The state of the world
Applications as declarative worlds where `y = 2 * x` works like a spreadsheet. Derived values are reactive pure functions — no side effects, no mutable state. Traces lineage to 1970s Reactive Programming, with early JS in Knockout.js (2010) and RxJS (2012).

### Signals: Push-based
A Signal is "an abstraction that represents a reactive value that can be read and modified." The basic implementation is a pub/sub pattern with eager notification. Key insight: signals using the push-pull algorithm don't dispatch state values — they notify of state change (cache invalidation, not value propagation).

### Computed: Pull-based
Computeds differ from signals in two ways:
1. They are lazy — invalidated when dependencies change, re-evaluated only when read
2. They auto-track dependencies — no manual dependency arrays

### The magic link (Auto-tracking & Cache System)
Uses a global `STACK` array holding `ComputeContext` for the currently executing computed. Two pieces per context:
- `setDirty` — marks this computed and its subscribers as dirty
- `addSource` — registers dependencies and stores cleanup functions for dynamic dependency graphs

Process: signal update → dirty flag → on read, push context to STACK → fn() execution reads signal getter → getter registers computed's setDirty as subscriber → addSource stores cleanup → dirty=false, pop STACK

### Full Implementation
~80 lines of TypeScript implementing signal() and computed() with the complete push-pull algorithm.

### Final flow
Push propagates invalidation eagerly; pull re-evaluates lazily. All setDirty calls are synchronous. Effect functions are "more to API design than to the algorithm itself."

## Sources Cited

- "How signals work" podcast w/ Kristen Maevyn and Daniel Ehrenberg
- "Reactivity" by Milo Mighdoll
- "Introducing Signals Preact" by Marvin Hagemeister and Jason Miller
- "Signal Boosting" by Joachim Viide
- "The evolution of signals in JavaScript" by Ryan Carniato
- "State-based vs Signal-based rendering" by Jovi De Croock
- "Push-pull functional reactive programming" by Conal Elliott
- Videos: "Beyond Signals" (Ryan Carniato), "Controlling Time and Space" (Evan Czaplicki)
- Libraries: knockoutjs, rxjs, solidjs-signals, preact-signals, alien-signals
- Reference implementation: https://github.com/willybrauner/signal-playground
- TC39 proposal-signals: https://github.com/tc39/proposal-signals (Stage 1)
