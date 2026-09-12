---
url: https://tomasp.net/blog/2015/library-frameworks/
title: "Library patterns: Why frameworks are evil"
author: Tomas Petricek
site: tomasp.net
date_published: 2015-03-03
date_fetched: 2026-08-08
topics:
  - software-engineering-craft
---

# Library patterns: Why frameworks are evil — Summary

Tomas Petricek argues that software developers should build **libraries** rather than **frameworks**, and provides concrete patterns for doing so in a functional style. The distinction is simple: a framework owns the control flow and calls your code; a library is called by your code, which stays in control.

Frameworks have three structural problems. First, they **don't compose** — two frameworks each demand to own the main loop, and there's no clean way to nest one inside another. Libraries, by contrast, can be called from the same program without conflict. Second, frameworks are **hard to explore** — you can't load one into a REPL and experiment with it interactively the way you can with a library. Third, frameworks **shape how you code** — they force you into inheritance hierarchies and mutable state that make the code harder to reason about and test.

Petricek offers five remedies. **Support interactive exploration** by designing libraries with an obvious entry-point type whose methods are discoverable through autocomplete. **Avoid complex callbacks** — higher-order functions are fine, but a function that takes multiple callback arguments that share state is a framework in disguise; split it into simpler, independent functions. **Invert callbacks with events and async** — rather than requiring the user to implement virtual methods, expose events and let them write their own control flow with async workflows. **Provide multiple levels of abstraction** — the high-level convenience function is fine as long as there's a lower-level, more explicit alternative one step beneath it. And **design for composition** — expose all the data other libraries would need to interoperate with yours.

The article is the second in a series on functional library design, following one on layers of abstraction. All examples are in F#, but the principles are language-agnostic.
