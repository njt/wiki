---
url: https://www.joshwcomeau.com/react/usememo-and-usecallback/
title: Understanding useMemo and useCallback
author: Josh W. Comeau
site: joshwcomeau.com
date_published: unknown
date_fetched: 2026-08-08
last_updated: 2025-12-03
topics:
  - software-engineering-craft
---

Josh Comeau's definitive explainer on React's two most misunderstood hooks. He builds from first principles — what a re-render actually is, why reference equality matters in JavaScript, and how `React.memo` interacts with both — to make the case that `useMemo` and `useCallback` are tools for two distinct problems: skipping expensive recalculations and preserving object/function references across renders to keep pure components pure.

The core insight: `useMemo` is a lil' cache with dependencies as the invalidation strategy. `useCallback` is syntactic sugar for `useMemo` applied to functions. Both exist to solve real performance problems, but Comeau is explicit that they shouldn't be applied preemptively everywhere — the React Profiler should drive their use, not superstition.

He identifies two legitimate preemptive use cases: inside generic custom hooks (where you don't know the future call site) and inside context providers (where a new object reference would force every consuming pure component to re-render). The article also models the more important skill: restructuring components to avoid the need for memoization entirely — pushing state down so unrelated state changes don't trigger expensive work.
