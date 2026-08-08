---
url: https://react.dev/reference/rules/components-and-hooks-must-be-pure
title: Components and Hooks must be pure
author: React.dev (Meta)
date_published: 2025
date_fetched: 2026-08-08
---

React's canonical reference on purity: the foundational rule that components and hooks must be idempotent, free of render-phase side effects, and must never mutate non-local values. The page covers what purity means in React's declarative model, why it matters for performance and correctness, and exactly where the boundaries sit — what's allowed (local mutation, lazy initialization) and what's forbidden (mutating props, state, hook arguments, or values after passing them to JSX).

The core framework: render is a calculation of the next UI, not a place for effects. React may call your component multiple times during rendering, so any code that runs during render must produce the same result given the same inputs. Side effects belong in event handlers (user-triggered) or Effects (post-render synchronization), never inline in the component body.

Specific rules covered: components must be idempotent (no `new Date()` or `Math.random()` during render); props and state are immutable snapshots — copy, don't mutate; hook arguments become immutable once passed (they may be used as memoization dependencies); and values passed to JSX must not be mutated afterward (React may eagerly evaluate JSX before the component finishes rendering).

The page is pragmatic, not purist. Local mutation during render is explicitly fine — creating an array and pushing into it inside the component body is allowed because the array is recreated fresh each render. Lazy initialization (`SuperCalculator.initializeIfNotReady()`) is acceptable as long as it doesn't affect other components.
