# Ship Your Interpreter (Basis)

Basis's agents wrote a formal semantics for an extended WHILE language and proved in Lean that the language's actual C interpreter — compiled to RISC-V — agrees with it on every program's output. "Shipping your interpreter" means treating the implementation people already run as ground truth, replacing test suites with a machine-checked refinement theorem, and getting verified tools almost for free on top.

---

## What the source argues

Verified software is becoming feasible because LLMs are getting good at theorem proving — but the bottleneck has moved to **specification**: how do you know the statement you proved captures behaviour you care about? Basis's answer is radical and elegant: don't write the spec by hand and hope it's right. Let the LLM write the semantics, then make the *same* LLM prove the semantics matches the real interpreter binary, instruction by instruction, on every possible program. A wrong rule means a proof obligation that won't close — and in their experience, the stuck obligation points at the wrong rule.

Key evidence:

- **Tests can't validate a semantics.** An early semantics passed every test yet disagreed with the binary when a function name hit 59 characters — the C `stringify` writes into a 64-byte buffer, so `snprintf` silently drops the closing `>`. No test thought to try that; real programs in dynamic languages rely on stringified functions.
- **The proof caught four semantic errors** (64-bit wrapping, `for`-initialiser abrupt status, print-vs-concatenation rendering, the 63-byte cut) and bugs in LLM-written C library lemmas (a `malloc` reserve no heap satisfied, a `realloc` lemma false under size wrap, an `exit` routine writing `errno` unmentioned).
- **Separation logic was the unlock.** A fixed-footprint spec for `malloc` is *provably impossible* — the LLMs proved in Lean that no fixed address set works, after a workaround attempt stalled for three weeks. Porting MachCSL to iris-lean, making the footprint depend on the heap, closed the rest of the proof in days.
- **Scale**: ~1M lines of Lean at peak (35 generated scripts, one rule per ~11,800 machine instruction), refactored to <200K; the refinement theorem alone ~140K lines over a month; four verified tools (~33K lines) opened as PRs in three days once the semantics was trusted.

## Key quotes

> "a semantics can pass every test and still be wrong"

The central claim, and the sharpest thing in the post. It generalises far beyond formal semantics: any generated artifact whose correctness is checked only against examples thought up by someone — human or agent — inherits exactly the blind spots of that imagination.

> "The C compiler's output, the C library and the memory allocator are all inside our theorem."

What makes this credible rather than a demo. The proof reaches from source-level semantics down through gcc's output, newlib and dlmalloc to raw RISC-V instructions — the trust base is the theorem statement plus two short definitions, not "we were careful."

> "Once the semantics is trusted, it *is* the language, and the binary we started from is no longer needed to run WHILE programs."

The compounding payoff: a validated specification becomes the interface everything else composes against, turning one month of hard proof into three days of cheap verified tooling.

## Themes

#concept #pattern #tool

## Opinionated take

This is the strongest articulation I've seen of the "verifier as ground truth" thesis in the agentic-coding discourse. It makes a move most verification talk avoids: instead of asking whether the agent's *output* is correct, it asks whether the agent's *specification of correctness* is correct — and then machine-checks the spec against reality. That is exactly the gap [[Agentic Verification]] gestures at with oracles and differential testing, but carried to its logical limit: the interpreter binary, not a test suite, is the oracle, and "every program" replaces "every test we thought of."

Two honest caveats. First, WHILE is a 1,855-line C interpreter; the jump to Lua and OCaml is asserted with a plan (cut the proof at the VM, exploit OCaml's self-hosting fixpoint) but not yet results — CPython with its decades of C quirks is a different universe of pain. Second, the economics are undisclosed: a month of agent compute for one refinement theorem is real money, and the post is silent on cost. Still, the architecture of the argument — spec as the product, proof as the feedback loop, tools as the compounding dividend — is a template worth taking seriously for anything verification-shaped.

## Related pages

This source strengthens [[Agentic Verification]] by giving its "cheap, unfakeable checking" thesis an existence proof at the extreme end: a proof-checked oracle that covers *every* input, not a sample. It nuances [[Specifications as the Product]] — here the specification is literally the durable artifact that makes all downstream tooling nearly free, but the twist is that the spec itself is only trustworthy because a second artifact (the refinement proof) vouches for it. It complicates [[Guardrails and Feedback Loops]]: the guardrail is not a linter or a review gate but a million lines of machine-checked mathematics, and the "loop" is the proof failure pushing the LLM back to revise a rule. And it sharpens the testing critique in [[A New Era for Software Testing]] with the 59-character-name example — the canonical case of a suite that was green and wrong.

---
*Sources: [[raw/ship-your-interpreters]], [[summary/ship-your-interpreters]]*
*Last updated: 2026-10-24*
