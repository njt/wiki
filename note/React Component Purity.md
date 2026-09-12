# React Component Purity

React's foundational contract: components and hooks must be pure functions of their inputs — idempotent, free of render-phase side effects, and never mutating values they don't own. This isn't a style preference; it's the architectural assumption that makes React's concurrent rendering, automatic batching, and selective hydration possible. Break it and you get bugs that are intermittent, hard to reproduce, and invisible in development.

---

## The Architecture Behind the Rule

React is declarative: you describe what the UI should look like given the current state, and React figures out how to get there. To do that efficiently, React reserves the right to call your component function multiple times — pausing, discarding, and restarting renders as needed. This is only safe if your component is a pure function.

> When render is kept pure, React can understand how to prioritize which updates are most important for the user to see first. This is made possible because of render purity: since components don't have side effects in render, React can pause rendering components that aren't as important to update, and only come back to them later when it's needed.

This is the deep reason purity matters. It's not about functional programming aesthetics — it's about giving the framework the freedom to optimize. If React knows your component has no observable side effects during render, it can throw away a render in progress and start over without anyone noticing. If your component writes to a global variable or mutates a shared object during render, that optimization becomes a bug factory.

The contrast with [[Signals — The Push-Pull Algorithm]] is instructive. Signals track mutation through a reactive graph: push invalidation, pull re-evaluation. React takes the opposite approach — eliminate the mutation tracking problem entirely by making renders pure calculations over immutable inputs. Two different mental models for the same fundamental challenge: how do you know what to update when state changes?

## What Purity Actually Means

> Components must always return the same output with respect to their inputs — props, state, and context. This is known as *idempotency*.

Three specific rules unpack this:

**Idempotent**: Same inputs → same output. `new Date()` and `Math.random()` are the canonical violations — they return different values every call. The fix is to move non-deterministic code into Effects (for synchronization) or event handlers (for user-triggered updates).

**No side effects in render**: Don't touch the DOM, don't write to global variables, don't send network requests from the component body. Event handlers and `useEffect` are the designated escape hatches. The heuristic: if it's at the top level of your component function, it runs during render.

**No mutation of non-local values**: Props and state are immutable snapshots. Hook arguments become immutable once passed (because hooks may use them as memoization dependencies). Values passed to JSX become immutable — mutating them afterward can produce stale UIs because React may eagerly evaluate JSX before the component finishes rendering.

## The Pragmatic Exceptions

The page is refreshingly unpurist. Several things that look like impurity are explicitly allowed:

> There is no need to contort your code to avoid local mutation.

Creating an array inside a component and pushing into it during render is fine — the array is recreated fresh each render, so the mutation isn't "remembered." This is a sharp contrast with the pattern of declaring the array outside the component, which *does* persist across renders and produces duplicated results.

> Lazy initialization is also fine despite not being fully "pure"

`SuperCalculator.initializeIfNotReady()` during render is acceptable as long as it doesn't affect other components. React doesn't demand strict functional purity — it demands idempotency. The distinction matters: `new Date()` breaks idempotency (different result every call); a one-time initialization that produces the same side effect every time is fine.

> As long as calling a component multiple times is safe and doesn't affect the rendering of other components, React doesn't care if it's 100% pure in the strict functional programming sense of the word.

This is the key insight. React's purity rule is an engineering contract, not a mathematical proof. The test is: can React call this function multiple times without observable difference in the output? If yes, you're pure enough.

## Immutability as the Default

The page treats immutability as a cascading rule through every React API surface:

- **Props**: Never mutate. `item.url = new Url(...)` is wrong; `const url = new Url(item.url, base)` is right.
- **State**: Never assign directly. `count = count + 1` is wrong; `setCount(count + 1)` is right. State variables are snapshots — direct mutation doesn't trigger re-renders.
- **Hook arguments**: Once passed to a hook, treat them as frozen. The hook may have used them as `useMemo` dependencies; mutating them afterward breaks memoization silently.
- **JSX-consumed values**: Don't mutate after passing to JSX. React may eagerly evaluate JSX expressions, so `styles.size = "small"` after `<Header styles={styles} />` can leave the header rendering stale styles.

The unifying principle is *local reasoning*: you should be able to understand what a component or hook does by looking at its code in isolation. Mutation breaks that — you have to track where else a value might be changed.

## Relevance to Agentic Development

This page is React documentation, not AI commentary. But it's relevant to the agentic development conversation for three reasons:

First, purity rules make code more predictable — and predictable code is what coding agents handle best. [[AI Slop Starts with the Codebase Itself]] argues that codebases speaking a dialect the models already know are productivity multipliers. React's purity rules are among the most heavily documented, widely discussed patterns in frontend development; agents trained on web development corpora know them cold.

Second, the deterministic enforcement angle from [[Guardrails and Feedback Loops]] applies directly: React's Strict Mode double-renders components in development specifically to surface purity violations. It's a linter built into the framework itself — deterministic detection of a structural rule violation, not a prompt pleading with the developer to please be careful.

Third, the constraint paradox: [[Constraint Decay]] shows that structural constraints degrade LLM backend performance by ~30pp. But React's purity constraints are different in kind — they *reduce* the solution space rather than expanding it. Telling an agent "make this component pure" eliminates entire categories of possible (but wrong) implementations. Some constraints make the problem harder; others make it easier. Purity rules are the latter.

---

## Key Themes

- **#concept Declarative UI as optimization substrate** — React's entire concurrent rendering architecture rests on the assumption that components are pure. The rule isn't moral; it's mechanical.
- **#pattern Immutable snapshots over mutable references** — Props, state, hook args, and JSX values all follow the same rule: once created/consumed by the framework, treat as frozen. Copy and replace, never mutate in place.
- **#concept Idempotency over functional purity** — React doesn't demand mathematical purity. It demands that calling your function multiple times with the same inputs produces the same output. Local mutation, lazy initialization, and other "impure" patterns are fine as long as they're idempotent.
- **#pattern Side effects have designated places** — Event handlers (user-triggered) and Effects (post-render synchronization). The component body is for calculation only.

## Critical Analysis

The page is excellent reference documentation — clear, example-driven, and honest about edge cases. Its strongest contribution is the idempotency-over-purity framing, which prevents the kind of functional-programming cargo-culting that makes people contort simple loops into `Array.map` chains for fear of mutation.

One gap: the page doesn't discuss what happens when purity rules interact with the broader React ecosystem. Server Components add a new dimension (async components that can read from databases during render — is that a side effect?). The `use` hook blurs the render/effect boundary by allowing promise resolution during render. These newer features sit uneasily with the purity model described here, and the page doesn't acknowledge the tension.

The section on hook argument immutability is the most practically useful and least discussed part of the page. The memoization corruption scenario — mutate a hook argument, call the hook again, get a stale memoized result — is a real production bug pattern that's hard to diagnose. But the fix (copy the argument) is so simple that the section could be twice as long and still earn its keep.

---

*Sources: [[raw/components-and-hooks-must-be-pure]], [[summary/components-and-hooks-must-be-pure]]*
*Last updated: 2026-08-08*
