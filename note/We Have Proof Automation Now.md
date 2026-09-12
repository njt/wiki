# We Have Proof Automation Now

Adam Langley (ImperialViolet) builds a Zstandard decompressor in Lean, using LLMs to handle the proof burden — and in doing so makes the case that dependent types have crossed from research curiosity to practical engineering tool. The bottleneck that kept formal verification in the lab (10× more time proving than coding, per seL4) dissolves when LLMs can generate correct proofs in ~20 minutes for pennies.

---

## Key quotes

> "With great type-system power comes great proof effort."

Langley's one-liner diagnosis of why dependent types stayed niche. The seL4 retrospective quantified it: 10× more time proving than designing, 20× more lines of proof than C. This ratio is what LLMs invert.

> "Proof irrelevance means that, once you've stated something that is correct, the contents of the proof don't matter — only its existence."

The property that makes LLM-generated proofs viable. A human-produced proof and an LLM-produced proof are equally valid to the type checker. You don't need to read or maintain the proof, just know it exists. This is the key insight Langley builds on, and it's correct — though it dodges the "proof engineering" problem of how proofs survive code changes.

> "We, practically speaking, have a new type of programming language available to us."

The thesis. Not "dependent types are cool" but "dependent types plus LLM proof automation is a new category of tool." Langley is cautious — noting his decoder is 10× slower than native zstd and that proof scaling in larger systems is unknown — but the claim is still striking.

> "Serving a complex and fickle god"

On F*'s SMT-solver-based approach to proof automation. The contrast with LLM-based proofs is instructive: SMT solvers require learning what pleases them; LLMs require learning how to prompt them. One is a formal system with predictable brittleness; the other is a stochastic system with unpredictable brittleness. Langley doesn't explore which is worse.

## Key themes

- **#concept** — **LLM-based proof automation**: The central contribution. Proof irrelevance means any proof that type-checks is valid, regardless of provenance. LLMs produce proofs that sometimes type-check. Ergo, LLMs are proof automation. This is a genuinely novel argument that sidesteps the reliability question entirely — a bad proof simply doesn't compile.

- **#concept** — **Proof irrelevance as architectural property**: Not just a type-theory curiosity but the property that makes the whole thesis work. In conventional programming, LLM-generated code must be read and understood by humans. In dependently-typed programming, LLM-generated proofs need only exist. This inverts the trust model: you trust the type checker, not the proof author.

- **#tool** — **Lean as engineering language**: Langley treats Lean as a programming language with a powerful type system, not as a theorem prover. His observations about strict evaluation, monadic `do` notation, and reference-count-based in-place mutation are the notes of someone building real software, not proving theorems. The "sharp edge" of missing linear types is exactly the kind of practical concern that formal-methods papers ignore.

- **#tool** — **Zstandard and FSE**: Langley's explanation of finite state entropy is the clearest short treatment available: more states than symbols, each state encodes (symbol, bits-to-read, baseline), and the trick is that multiple states per symbol let you average fractional bits. The backwards-decoding constraint is a genuine algorithmic oddity, and his point that you already need the whole sequence for probability computation anyway is the kind of practical observation that makes constraints manageable.

- **#pattern** — **Type system as comment replacement**: The argument that invariants encoded in types persist while invariants in comments rot. This isn't new (it's the dependent-types pitch since forever), but Langley's framing — "arbitrarily subtle invariants that, in a conventional language, end up as comments and get lost as teams grow" — makes it concrete. The four FSE properties he proves are exactly the kind of thing that would be a comment in a C implementation, and exactly the kind of thing that rots.

## Critical analysis

**The argument works, but the caveats are bigger than Langley admits.** His decoder is 10× slower than native zstd. He notes this but doesn't dwell on it. For decompression — a performance-critical operation where zstd's whole value proposition is speed — a 10× slowdown makes the exercise academic. The right comparison isn't "verified Lean vs. unverified C" but "verified Lean vs. verified C using conventional methods," and Langley doesn't make that comparison. The seL4 team proved a C implementation correct; they didn't need Lean.

**The proof-engineering problem is real and unaddressed.** Langley mentions that proofs must be structured to survive code changes, then moves on. But this is the whole game. If you change the FSE table construction and all four proofs break, and you need to regenerate them with an LLM, you're back to the same maintenance burden — just with a different tool. The question isn't "can LLMs generate proofs?" but "can LLMs generate maintainable proofs?" and Langley's experience — the LLMs needed to change his code style away from imperative `Id.run` — suggests the answer involves constraining how you write code, not just how you write proofs.

**The refusal to publish code is telling.** Langley says LLMs can "probably do better than he did" for small, well-defined cases. This inverts the usual open-source dynamic: normally you publish so others can improve. Here, the improvement mechanism (LLM generation) makes the reference implementation less valuable. If the code is the spec and the proof is the verification, and both can be regenerated cheaply, what's the durable artifact? The type signatures. This is [[The Oracle Is the Asset]] applied to formal methods: the spec (types) is the durable asset; the implementation and proof are disposable.

**The "new type of programming language" claim is premature but directionally correct.** Langley is right that something has changed. The combination of proof irrelevance + LLM generation eliminates the bottleneck that killed dependent types for practical work. But we don't know whether the result is a usable language or a laboratory curiosity with better tooling. The 10× performance gap, the scaling question, and the proof-maintenance problem are all real. What Langley has demonstrated is a proof of concept — literally and figuratively.

**The verified assembly aside is the most interesting undeveloped thread.** If you can prove equivalence between a high-level verified implementation and optimized assembly, you get correctness plus performance. Langley couldn't make it scale, but the architecture is right: use Lean for specification and verification, use `extern` to call native code at runtime, and prove the two equivalent. This is the escape hatch from the 10× performance penalty, and it's where the real engineering work lies.

**Why this matters more than it seems:** The combination Langley demonstrates — dependent types + LLM proof generation — is a specific instance of a general pattern: LLMs making previously impractical formal methods practical. If proofs are cheap, what else becomes cheap? Verified protocols, verified parsers, verified crypto — the entire class of software where correctness matters more than raw performance. The performance gap matters less when you're verifying a TLS handshake than when you're decompressing gigabytes.

---

## Related pages

- [[The Oracle Is the Asset]] — Specs as durable artifacts; same dynamic applied to types and proofs
- [[Lean Software Scaling Laws]] — Gwern's hypothesis about formally-strong languages having better scaling exponents
- [[Software Engineering Craft]] — The fundamentals that don't change, even when proof automation does
- [[Constraint Decay]] — Formal properties of code that agents struggle with; the mirror image of what Lean enforces
- [[An Introduction to Formal Logic (Peter Smith)]] — The logical foundations: Smith's textbook is the standard introduction to the first-order logic that proof assistants embody; useful for understanding *what* the machine is checking
- [[Guardrails and Feedback Loops]] — The type checker as the ultimate deterministic guardrail
- [[AI Code Migration with Claude Code]] — Anthropic's approach to large-scale code transformation; related to proof maintenance under code change
- [[Why Rocq Is Better Than Lean for Program Verification]] — Korkut's ecosystem-side counterpoint: Langley shows Lean proofs are now cheap to generate; Korkut shows that Rocq's two decades of verification infrastructure (CompCert, Iris, VST) is what you'd need to rebuild if you switched

---
*Sources: [[raw/zstd-lean]]*
*Last updated: 2026-07-29*
