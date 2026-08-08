# React useMemo and useCallback

Josh Comeau's definitive mental-model builder for React's two most misunderstood performance hooks: what they do, why reference equality makes them necessary, when to reach for them, and — crucially — when restructuring your components is the better answer.

---

## Key Quotes

> "useMemo is essentially like a lil' cache, and the dependencies are the cache invalidation strategy."

The one-sentence explanation that lands better than the React docs. It frames memoization not as a React-specific magic trick but as a general computer science concept (cache with invalidation) applied to component renders. This is the line to remember when explaining these hooks to someone who's struggling.

> "Every time React re-renders, we're producing a brand new array. They're equivalent in terms of value, but not in terms of reference."

The second use case — preserved references — is the one that catches experienced developers off guard. Comeau's explanation of why `React.memo` wrapping a component doesn't protect it when its props include inline-created arrays, objects, or functions is the article's most transferable insight. The JavaScript detour (two arrays with identical contents are not `===`) is the right pedagogical choice — this isn't a React problem, it's a JavaScript-semantics problem that React happens to surface.

> "useCallback is syntactic sugar. It exists purely to make our lives a bit nicer when trying to memoize callback functions."

The demystification move. `useCallback(fn, deps)` is exactly `useMemo(() => fn, deps)`. There's no additional mechanism, no separate optimization path. Comeau shows the equivalent `useMemo` version, then introduces `useCallback` as the ergonomic shorthand. This is the right order — show the mechanism first, then show the sugar.

> "The best way to use these hooks is in response to a problem. If you notice your app becoming a bit sluggish, you can use the React Profiler to hunt down slow renders."

The disciplined default. Comeau explicitly pushes back against the reflex to wrap everything "just in case." His two preemptive-use exceptions (generic custom hooks, context providers) are narrow and justified — not because memoization is always good, but because those specific cases have unknown future consumers whose performance you can't profile yet.

> "We hear a lot about lifting state up, but sometimes, the better approach is to push state down!"

The restructuring alternative that's more important than any hook. In the prime-number example, extracting `Clock` and `PrimeCalculator` into separate components achieves the same performance result as `useMemo` — but by changing the architecture rather than papering over it. This is the senior-engineer move: fix the design so the problem doesn't exist, rather than optimizing a design that creates unnecessary work.

---

## Key Themes

- **#pattern** **Memoization as cache-with-invalidation** — Comeau frames `useMemo` in general CS terms rather than React-specific ones: a function whose result is cached, with a dependency array as the cache-busting strategy. This framing makes the hook legible to developers coming from any language.

- **#concept** **Reference equality as the hidden constraint** — The article's most valuable section explains why `React.memo` fails silently when props are recreated on every render. This is JavaScript's `===` semantics interacting with React's render model, and understanding it is table stakes for React performance work.

- **#pattern** **Push state down before you memoize** — The restructuring alternative. If two unrelated pieces of state live in the same component, changes to one force recomputation of the other. Extracting each into its own component eliminates the problem without any hooks. This is the architectural instinct that `useMemo` can't replace.

- **#concept** **Profile first, optimize second** — Comeau's meta-advice: use the React Profiler to find actual bottlenecks before reaching for `useMemo` or `useCallback`. Most re-renders aren't expensive enough to matter, and premature memoization adds complexity without benefit. This aligns with the classic performance engineering wisdom (see [[Performance Optimization Loop]]).

- **#tool** **React.memo as component-level memoization** — The pure component wrapping that protects against parent re-renders when props haven't changed. Comeau shows that it only works when props preserve their references, which is why `useMemo` and `useCallback` exist: they're the prop-stabilization layer that makes `React.memo` effective.

---

## Critical Analysis

**The article's greatest strength is its pedagogy, not its novelty.** None of the information is new — the React docs cover all of this. But Comeau's explanation order (re-renders → heavy computation → reference equality → `useMemo` → `useCallback` → when to use) is carefully constructed to build the mental model one layer at a time. Each concept motivates the next. The interactive diagrams (snapshot stacks, re-render graphs, reference-preservation sketches) do the heavy lifting that prose alone can't. This is how technical writing should work: build the intuition, then name the thing.

**The "push state down" advice is more important than the hooks themselves**, and it gets less space in the article than it deserves. In a large codebase, architectural restructuring eliminates entire categories of performance problems, while `useMemo` eliminates individual instances. Comeau acknowledges this ("this won't always be an option") but doesn't fully explore the tension: in practice, developers reach for hooks because restructuring requires touching multiple components and convincing teammates, while `useMemo` is a one-line local change. The hook is organizationally cheaper; the restructuring is architecturally better. The article could be stronger on when to fight for the restructuring.

**The context-provider use case is under-explained.** Comeau says to memoize context values because "there might be dozens of pure components that consume this context," but doesn't walk through the failure mode: parent re-renders → new context object → every `useContext` consumer re-renders, even if their slice of the context value hasn't changed. This is the single most impactful `useMemo` in a typical React app, and it gets one paragraph. The omission is understandable for article length, but a practitioner who remembers only the prime-number example will miss the pattern that matters most in production.

**The relationship to signals is the elephant in the room.** The article was last updated December 2025, and the React ecosystem in 2026 is increasingly signals-aware. Signals (see [[Signals — The Push-Pull Algorithm]]) eliminate the reference-equality problem entirely — they auto-track dependencies rather than requiring manual dependency arrays. Comeau's article explains *why* manual dependency management is necessary in React's current model, which makes it an unintentional argument for why signals are winning. When you read "we list `boxWidth` as a dependency, because we do want the `Boxes` component to re-render when the user tweaks the width," you're reading a description of work that signals do automatically. The article is the best explanation of the problem that signals solve.

**The "useCallback is syntactic sugar" framing is correct but incomplete.** Yes, `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`. But the sugar matters: the `useCallback` version avoids an extra function allocation on every render (the arrow wrapper in the `useMemo` version is recreated each time). In most cases this is negligible, but in the hot-loop scenarios where you're reaching for these hooks, it's not nothing. Comeau acknowledges this implicitly by calling it syntactic sugar "to make our lives a bit nicer" — but nicer ergonomics often correlate with better performance when the alternative involves allocating and discarding wrapper functions.

**The omission of `useRef` as an alternative is notable.** For the "preserved reference" use case where the value never changes (empty dependency array), `useRef` with a lazily-initialized value is sometimes the better choice — it guarantees stable identity without the overhead of dependency tracking. This is a niche case and probably outside the article's scope, but a practitioner who only knows `useMemo` for stable references will over-use it.

**The AI angle is absent but relevant.** The article predates the era where AI coding agents routinely generate React code. But reading it in 2026, the pedagogical clarity is a control surface for [[What Frontend Developers Still Hate — 2026 Survey|the specific AI failures frontend developers report]]: AI-generated code violates the rules of hooks, creates unnecessary `useState` calls, and places state outside components. Comeau's article is the mental model AI doesn't have — the understanding of *why* these hooks exist that would prevent an agent from wrapping every value in `useMemo` or, conversely, from never using them when a context provider desperately needs one.

---

## Related

- [[What Frontend Developers Still Hate — 2026 Survey]] — The survey where developers report AI getting hooks wrong; Comeau's article is the corrective mental model
- [[Signals — The Push-Pull Algorithm]] — Signals auto-track dependencies; useMemo/useCallback are React's manual equivalent, and the contrast explains why signals are winning
- [[Performance Optimization Loop]] — Gordon's measurement-driven methodology; Comeau's "profile first, use hooks in response to data" is the same discipline applied to React
- [[Performative UI]] — Satirical React component library; the same component model where these hooks operate
- [[Constraint Decay]] — AI's failure mode when structural constraints accumulate; React's rules of hooks are the frontend equivalent of the database/ORM constraints that break agents

---
*Sources: [[raw/usememo-and-usecallback]], [[summary/usememo-and-usecallback]]*
*Last updated: 2026-08-08*
