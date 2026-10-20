# Design by Contract and Effects for LLM-Generated Code

Gavin Ray's short, code-first essay argues that Design-by-Contract and explicit effects — two decades-old language features — jump roughly "tenfold" in value when code is authored by LLMs, because they turn intent and operational behaviour into machine-checkable artifacts. The payoff he points at is compiler-generated reports of *semantic* changes in a PR: effects added, postconditions weakened. Written as pseudo-syntax examples plus a thesis, with no working toolchain behind it.

---

## What it argues

The development loop Ray describes is the now-standard one: prompt an LLM for an implementation, ask it to write tests, run them, poke the app, review if there's time. His premise is that in this loop, "code that verifiably does what it says on the tin" is worth vastly more than in human-authored development — because the human's residual role is verification, and verification needs something to verify against.

- **Contracts** put preconditions, postconditions and invariants in the signature itself. A `User` struct with `invariant email.is_valid_email()` and a `change_email` that `ensures user.email_verified == false` mechanically catches the exact bug class LLMs produce: implementations that satisfy the letter of the request while missing the implied semantics.
- **Effects** make operational behaviour visible at the call site. `load_settings` reading the filesystem and `parse_settings` parsing an in-memory string return the same type but "have meaningfully different OPERATIONAL behavior. That difference should be visible at the call site." An `encode` function without the `alloc` effect physically cannot create a temporary array.

The synthesis is the PR report: not "what lines changed" but what the contract surface changed. That is the level at which a human who no longer reads the diff can still pass judgement.

---

## Key quotes

> "The majority of the code I have 'written' (prompted) and shipped in the last year was not authored by me."

The disclaimer matters too — "No part of the prose was machine-generated. You will not find machine-written prose on this blog. I consider it deeply disrespectful" — a practitioner writing about machine code while staking identity on human prose.

> "These functions may return the same TYPE, but they have meaningfully different OPERATIONAL behavior."

This is the sharpest observation in the piece, and it's one static analysis has known for decades. Types describe values; effects describe what running the function *does to the world*. For an agent that writes plausible-looking code with the wrong I/O shape, the signature is the first line of defence.

> "Postcondition weakened: Ledger.append — ensures ledger.length == old(ledger.length) + 1"

The dream artifact: a machine-readable diff of *semantics*, not text. "Weakened" is the crucial word — a postcondition that got weaker means the function now promises less, which is precisely the change a reviewer of generated code most needs to see and almost never sees in a diff.

---

## Themes

#concept #pattern #tool

---

## Analysis

Ray's argument is a language-design corollary of the verification turn this wiki keeps circling. Where [[The Coming Need for Formal Specification]] argues the bottleneck moves upstream to *specification* and reaches for TLA+/Lean-grade formal methods, Ray makes the lighter-weight case: you don't need full proofs, you need contracts and effects as ordinary parts of signatures, so the compiler can do the checking. That's the pragmatic middle Congdon's essay is accused of skipping — contracts are exactly the "layered correctness" middle ground. The two pieces also expose a gap: neither says who writes the contract. If the LLM writes both the code and its postconditions, the contract inherits the same trust problem as the code — a human still has to judge whether `ensures ledger.length == old(ledger.length) + 1` is the *right* promise. DbC shifts review from code to specification, it doesn't eliminate it.

There's a natural alliance with the agentic property-based testing line of work. [[Finding Bugs with Property-Based Testing]] and [[Agentic Property-Based Testing]] show LLMs are good at *inferring* properties (from docstrings, types, call sites) and letting Hypothesis hunt counterexamples. Contracts would make those properties explicit and checkable at compile time instead of inferred — the same division of labour, moved left in the toolchain. And [[Constraint Decay]] found that explicit, structural constraints *reduce* rather than degrade agent performance (React purity rules being the counterexample), which is direct empirical support for Ray's premise that agents do better when the rules are visible in the type system rather than ambient in the culture.

The honest weakness of the piece is that it's aspirational. No mainstream language ships both first-class DbC and an effect system (Dafny, Ada SPARK and Koka each have one half); no toolchain produces his PR reports today. If [[Lean Software Scaling Laws]]'s prediction is right that formally-strong languages become more LLM-predictable at scale, the demand side of Ray's argument strengthens — but the supply side (a language with both features that anyone actually uses) is still missing. Meanwhile the effect half has a second life Ray doesn't mention: effect declarations are a natural **permission model for agents** — an agent sandbox can mechanically refuse calls whose effect set exceeds its grant, connecting this to sandboxing rather than just to review.

---

*Sources: [[raw/design-by-contract-and-effects-for-llms]], [[summary/design-by-contract-and-effects-for-llms]]*
*Last updated: 2026-10-20*
