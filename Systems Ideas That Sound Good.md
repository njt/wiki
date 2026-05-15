# Systems Ideas That Sound Good

Steven Sinofsky's catalogue of engineering patterns that sound appealing in theory and fail 9 out of 10 times in practice. Hard-won wisdom from decades at Microsoft, delivered with the authority of someone who watched (and sometimes shipped) every one of these mistakes.

---

## Key Quotes

> "The only way something is truly pluggable is if a second implementation is designed at the exact same time as the primary implementation. Then at least you have a proof it can work... one time."

> "9 out of 10 times once you go outside a framework or the data layer and think you can manage asynchrony yourself, you'll do great except for the bug that will show up a year from now that you will never be able to reproduce."

> "Fail as you diverge from the underlying platform or as you build capabilities that are expressed wildly differently on each target."

## Key Themes

#simplicity #api-design #cognitive-debt

Eight traps, each seductive:

1. **Pluggability** -- the API is the behavior, not the header file. You need two implementations at design time to prove it works.
2. **Platform via APIs** -- adding APIs doesn't make you a platform. Partners have their own businesses.
3. **Over-abstracting** -- premature abstractions rot unused; post-hoc abstractions strangle performance and security.
4. **DIY async** -- frameworks handle it; you won't, and the bug shows up in a year.
5. **Deferred security** -- addressing access control post-launch guarantees failure.
6. **Data sync** -- remains genuinely hard; companies worth billions exist for this reason alone.
7. **Cross-platform** -- you're building an OS. Microsoft forked Office in 1998 rather than maintain unity.
8. **Escape to native** -- frameworks maintain state that native calls bypass.

The common thread: each pattern feels like it simplifies but actually shifts complexity to where it's less visible and harder to debug. This is [[Simplicity in the Age of AI-Assisted]] applied to architecture decisions -- the inherited complexity that LLMs won't question because they were trained on codebases that already made these mistakes.

The pluggability point connects to [[Spec-First Development at Benchling]] -- Benchling's unified contract is the rare case where pluggability works, because they designed the platform and the first consumer simultaneously.

## Critical Analysis

Sinofsky writes from the position of someone who shipped Windows and Office at scale, which gives the prescriptions unusual weight. These aren't theoretical objections; they're post-mortems.

The weakness is that it's from a particular era (desktop + OS platform). Some of these patterns have become more tractable since (cross-platform via web, async via language-native constructs like Go goroutines or Elixir processes). But the core principle -- that familiar patterns are seductive precisely because they defer complexity -- is timeless. The fact that [[Loomkin]] chose Erlang/OTP specifically to handle concurrency is a direct nod to Sinofsky's async warning.

---
*Sources: [[raw/systems-ideas-that-sound-good]]*
*Last updated: 2026-05-14*