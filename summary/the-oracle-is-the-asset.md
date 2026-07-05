---
url: https://intertwingly.net/blog/2026/06/12/The-Oracle-Is-the-Asset.html
title: The Oracle Is the Asset
author: Sam Ruby
date_fetched: 2026-06-15
date_published: 2026-06-12
---

# The Oracle Is the Asset

By Sam Ruby (intertwingly.net), June 12, 2026.

## Summary

Sam Ruby argues that web frameworks are about to flip from runtime libraries to transpilers, and when they do, the durable asset won't be the compiler — it'll be the test suite ("the oracle") that defines correct behavior. He uses his own Roundhouse project as an existence proof: a retrofit compiler for Rails targeting nine languages, built by one retired person in weeks with Claude Code. Ruby himself can't write a compiler — he wrote the oracle, and AI generated whatever satisfied it. This is a Drucker inversion: "you manage what you measure" becomes "you own the spec the compiler answers to; the implementation is regenerable."

## Key Claims

1. **The economic inversion.** Compiler construction was prohibitively expensive; now it's cheap. The scarce, durable asset shifts from implementation to reference behavior.

2. **Transpilers, not LLVM backends.** What got cheap is writing the compiler, not owning a GC, scheduler, standard library, or debugger. Source-to-source compilers borrow all of that from existing toolchains per target.

3. **Downstream of incumbents, not replacing them.** LLMs are fluent in existing frameworks; this makes incumbent syntax the natural compilation source format. The dull cause (users are on the incumbent) sets the direction; the LLM cause lowers the cost and deepens the lock-in.

4. **The oracle is three layers:**
   - Fixed-value model/controller tests (runs natively per target, no Rails dependency)
   - Compare tests against live Rails (DOM diff catches structural drift)
   - Browser e2e tests via Playwright (dynamic behavior like Turbo Streams, WebSockets)

5. **The Drucker inversion.** "I didn't write Roundhouse. I can't write a compiler." He wrote the oracle, and Claude Code wrote whatever satisfied it. The principal directs by outcome; the outcome is the oracle.

6. **Partiality becomes shippable.** A runtime library missing a method fails in production. A compiler missing a method refuses at build time with a location. Framework compilers can ship at every point on their coverage curve.

## Three Interpretive Jobs

Ruby identifies three roles the framework interpreter performs:
- **Production** → goes to emitted targets (the performance story)
- **Feedback loop** → never belonged to interpretation; modern JS toolchain (HMR, 47ms system tests) already beats the interpreter at its claimed strength
- **Semantic authority** → what remains with the reference implementation: the referee every target is tested against

## Boundaries and Falsifiers

Where the bet stops: request-invariance (compilation wins where decisions can't differ between requests), carrying cost (framework semantics land once, per-target emitters stay thin), and where the test suite stops asserting.

Six things that would prove the thesis wrong, and honest admissions about the corpus gap (frequency weights come from a handful of blog-shaped apps) and the governance question (answered by JRuby precedent).

## Closing

"A compiler I could not have written, built to pass an oracle I could." Two claims braided: an economic one (cheap compilers make the oracle the scarce asset) and a methodological one (the oracle is what the principal authors; implementations are generated to satisfy it).

Roundhouse is open source under MIT / Apache-2.0 dual license.
