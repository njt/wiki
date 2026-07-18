# The Oracle Is the Asset

Sam Ruby's argument that web frameworks are about to flip from runtime libraries to transpilers, and when they do, the durable asset won't be the compiler — it'll be the test suite that defines correct behavior. The economic inversion is downstream of AI making compiler construction cheap: once implementations are regenerable, the spec (the "oracle") is the thing worth owning. Ruby's Roundhouse project — a Rails-to-nine-languages transpiler built by one retired person with Claude Code in weeks — is the existence proof.

---

## The Core Argument

Ruby identifies a Drucker inversion in software: "what you measure is what you manage" becomes "you own the spec the compiler answers to; the implementation is regenerable." He didn't write Roundhouse because he can't write a compiler. He wrote the oracle — three layers of tests that define correct behavior — and Claude Code generated whatever satisfied it.

This reframes the durable asset claim: the oracle isn't durable merely because implementations are cheap to regenerate. It's durable because **it's the thing that was written.** The principal directs by outcome, and the outcome is the oracle.

## Key Quotes

> "Frameworks currently operating as runtime libraries will increasingly become transpilers. When that shift occurs, the durable asset turns out to be the reference behavior they're tested against, not the implementations that produce it."

The thesis in one sentence. This is the same inversion [[The Dark Factory is a DOT File]] and [[Specifications as the Product]] describe, applied specifically to frameworks. The difference: Ruby argues this is about to happen to *your existing framework*, not just greenfield tools.

> "I didn't write Roundhouse. I can't write a compiler; I've never written one. I wrote the oracle — fixtures, test layers, compare gate, framing of which Rails subset is fair to target — and Claude Code wrote whatever satisfied it."

The Drucker inversion in practice. This is the methodological claim paired with the economic one: cheap compilers make the oracle the scarce asset, *and* the oracle is what the principal authors. Implementations are generated to satisfy it.

> "The headline benchmark — emitted Ruby on JRuby at 54× — was run for the first time immediately before posting, not from hand-written confidence but because the oracle had already verified everything. The first-person confirmation was redundant by construction."

A genuinely striking piece of evidence. The 54× number wasn't arrived at by hope or manual verification — it was already proven by the test suite before the author ever ran it. This is what it looks like when the oracle is the asset.

> "A runtime library lacking a method fails in production; a compiler lacking a method refuses at build time with a location. This makes partial framework-compilers shippable at every point on their coverage curve."

The partiality insight. A traditional compiler needed near-complete coverage to be useful; a framework compiler that stops at build time on uncovered methods can ship at 20%, 50%, 80% — each increment is usable. The 80/20 point "gets discovered by compilation instead of estimated by survey."

## The Oracle's Three Layers

Each layer fails in a different direction:

1. **Fixed-value model/controller tests.** Minitest-style, written into the source, compiled alongside, running natively per target without referencing Rails at all. Ruby calls this the floor — the layer he'd most want a skeptic to examine.

2. **Compare tests against live Rails.** Fetch the same URL from Rails and a target, canonicalize both DOM trees, diff. Catches structural drift the fixed tests didn't anticipate. This is the bridge layer — verifies the compiler isn't subtly wrong where both implementations produce "valid" output that differs.

3. **Browser e2e tests via Playwright.** Dynamic behavior — Turbo Stream inserts, Action Cable broadcasts, validation re-renders, computed Tailwind — that static DOM diff cannot reach. This is the ceiling for behavior that matters to users.

This three-layer design is the architectural insight: no single test type is sufficient, but their union covers different failure modes with different cost profiles.

## Key Themes

#spec-driven #compiler #oracle #testing #ai-generated-code #framework #transpiler

## Critical Analysis

**This is the most important spec-as-asset argument yet written**, and it's important precisely because it comes from outside the AI-hype circuit. Ruby is a twenty-year open-source veteran (Apache board, W3C, WHATWG, Rails core contributor in the early days). He's not selling anything. Roundhouse is MIT/Apache-2.0 dual-licensed. The piece is careful about where observation ends and inference begins.

**The Drucker inversion is the freshest idea here.** We've been saying "specs are the product" for months — [[Specifications as the Product]], [[The Dark Factory is a DOT File]], [[Write Only Code]]. Ruby adds something new: the spec isn't just the durable artifact; it's **the only thing the human wrote.** The compiler wasn't hand-coded. It was generated to satisfy the oracle. This inverts the relationship between author and artifact more radically than anything in the spec-as-product literature.

**The "partiality becomes shippable" insight is underappreciated.** Traditional compilers had a coverage cliff — you needed ~100% of the language spec before anyone could use it. Ruby's point that a *framework* compiler can ship at any coverage percentage because it fails at build time (not in production) is obvious in retrospect and important in practice. It means the barrier to entry for framework compilation isn't completeness — it's "enough to cover the conventions your app actually uses." [[Malloy]] applies this same inversion to SQL: the semantic model (measures, dimensions, joins) is the durable oracle, and the SQL is a regenerable artifact — compiled fresh for each target dialect, safe to throw away because the model can reproduce it on demand.

**The Haxe comparison is telling.** Ruby invokes Haxe as the ghost: it proved multi-target transpilation viable fifteen years ago and stayed niche. His diagnosis: Haxe had thin emitters and shared semantics, but lacked "a convention-rich single source framework underneath it." The framework supplies the spec, the dependence analysis, and the oracle for free; the language supplies none of them. This is a real structural argument, not hand-waving.

**Where the argument is weakest:** The corpus gap is honestly admitted but substantial. Ruby's frequency weights come from "a handful of apps biased toward blogs and 37signals idiom." For a claim this broad — that *your* framework compiler is affordable *now* — the corpus is the bottleneck. More apps fix the ranking but not the selection bias. And the governance question (how does a downstream compiler track an upstream framework?) gets the JRuby precedent as an answer, which is plausible but lighter than the rest of the argument deserves.

**The six falsifiers are the mark of intellectual seriousness.** Ruby doesn't just state conditions that would prove him wrong; he states conditions that would prove the *opposite* — implementations as the durable asset, WASM as the universal target, bespoke LLM apps as the maintainable default. The piece invites disconfirmation rather than dodging it.

**What this means for the wiki's existing spec-as-product thesis:** Ruby's argument strengthens [[Specifications as the Product]] by adding a concrete mechanism — the oracle-as-compiler-target — and an existence proof. It also sharpens the thesis: it's not just that specs outlive code; it's that in the limit, the spec *is* the code, and the code is a transient artifact generated from the spec by an AI that the principal couldn't have used for the same task five years ago.

## Connections

- [[Specifications as the Product]] — the same economic inversion, synthesized across multiple sources. Ruby adds the compiler-target mechanism and the existence proof.
- [[The Dark Factory is a DOT File]] — "software is cheap now, specs are the expensive part." Ruby's oracle is the DOT file for framework compilers.
- [[Guardrails and Feedback Loops]] — the three-layer oracle is a guardrail architecture applied to compiler correctness.
- [[The Claude C Compiler]] — another AI-generated compiler as existence proof, though at the language level rather than the framework level.
- [[Write Only Code]] — the logical conclusion: if the oracle is the asset, the generated code is write-only.
- [[Spec-Driven Development]] — the spec-code-test triangle; Ruby's oracle is the test vertex wired to compiler verification.
- [[Compound Engineering]] — the 50/50 rule; Ruby invested in the oracle infrastructure, not the compiler code.
- [[FrontierCode]] — benchmarks as oracles; the mergeability benchmark is a spec for "what good code looks like."

---
*Sources: [[summary/the-oracle-is-the-asset]]*
*Last updated: 2026-06-15*
