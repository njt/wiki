# Why Rocq Is Better Than Lean for Program Verification

Joomy Korkut's detailed, code-backed argument for why Rocq (née Coq) remains a better fit than Lean for *program* verification — not math formalization, where Lean's momentum is real. The post makes its case across three axes: language-level support (coinductive types, nested inductives), program extraction (multiple backends vs. Lean's single runtime pipeline), and ecosystem depth (two decades of verification infrastructure that a weekend port can't replicate). The subtext is a question every tools person eventually faces: when does accumulated infrastructure outweigh the hype cycle?

---

## Key quotes

> "I am not trying to start a flame war here. If you are a more mature person than I am, feel free to mentally replace 'better' with 'a better fit for my work today' everywhere below."

The disclaimer that preempts the flame war the post is absolutely trying to start. Korkut knows exactly what he's doing — the original slide was provocatively titled — but the caveat is honest: this is a "fit for my work" argument, not a universal ranking. The distinction matters because his work (interaction trees, game trees, verified games extracted to C++) leans hard on features Lean doesn't have.

> "Lean does not hand you a menu of alternate extraction backends, and the compilation pipeline it has offers no end-to-end correctness proof."

The most pointed critique of Lean's extraction story. Rocq offers OCaml, Haskell, Scheme, Rust, Elm, C++, Clight, and WebAssembly — plus a verified extraction pipeline. Lean gives you one runtime with impressive performance (lean-zip beats Rust's miniz_oxide!) but no verified pipeline and no readability. Korkut's framing: performance alone isn't the whole story; you also need to read the code, choose the target, or put it through a verified pipeline.

> "With Lean, I have to give up one of these things or rebuild it on top of generic machinery. Rocq lets me declare codata, write a guarded producer, reason about it by observation, and extract direct lazy code."

The thesis of the coinduction section, backed by runnable counterexamples. Korkut tests QPFTypes — the closest Lean experiment to Rocq's `CoInductive` — against three ordinary Rocq patterns: parameterless types, mutual coinduction, and indexed families. All three fail. The Lean alternatives (`Stream'`, `Iter`, `Thunk`, `partial def`) each solve one piece but lose either proof transparency, direct extraction, or the ability to model branching continuations (interaction trees). This is the section that will age fastest — Lean could fix any of these — but as of mid-2026, it's correct.

> "You might skim that list and think, 'None of this is a big deal; I can vibe-port the piece I need to Lean over a weekend.' Maybe you can. But hold that thought for one more section."

The transition into the ecosystem argument. The list preceding this line — interaction trees, Iris, CFML, Perennial, VST, CompCert, Vellvm, Fiat Crypto, and a dozen more — represents accumulated person-centuries of verification engineering. Korkut's point isn't that any single item is irreplaceable; it's that the *set* is. Each one you port is a weekend you're not doing your actual work, and they keep accumulating while you port.

> "'But the AI only knows the popular language' is an argument with a short shelf life, and it is not a reason for me to switch proof assistants."

A cleanly delivered rejoinder to the most common question Korkut gets. The 2024 argument — "AI writes better Lean because there's more training data" — made tactical sense when models were brittle. In 2026, models adapt to unfamiliar languages when given documentation. Rocq's 35-year corpus gives it a training-data advantage that compounds, and the argument's shelf life was always bounded by model capability.

---

## Key themes

- **#tool** — **Rocq vs. Lean as program verification tools**: The post is a side-by-side comparison at three levels: language features (codata, nested inductives), extraction pipeline, and ecosystem. Korkut is clear about scope: this is about *program* verification, where Rocq's coinduction and extraction story matter; Lean may genuinely be better for math.

- **#concept** — **Coinductive types as the differentiator**: The deepest technical section. Rocq's native `CoInductive` / `CoFixpoint` support gives you: (a) kernel-checked guardedness for recursive producers, (b) direct lazy extraction to OCaml, and (c) proof by observation. Lean's alternatives (`Stream'`, `Iter`, `Thunk`, `partial def`, QPFTypes, library-encoded M-types like PolyFun) each sacrifice one of these. For interaction trees — branching, effectful, possibly nonterminating programs — only native codata gives you all three.

- **#pattern** — **Nested inductive acceptance as a language-design choice**: The JSON schema validation example reveals a genuine gap: Lean's kernel rejects `Forall₂` + `And` nesting that Rocq accepts. Korkut shows a workaround (splitting into two `Forall₂` derivations) but notes it loses the single proof object pairing names with validations. The deeper point: both languages are conservative about what inductives they accept; they just draw the line in different places, and Rocq's line includes more of the patterns that show up in program verification.

- **#pattern** — **Extraction as a menu vs. extraction as a pipeline**: Lean gives you one fast pipeline to its own runtime. Rocq gives you a dozen targets, some verified, some readable, trading off TCB size vs. performance vs. ergonomics. Korkut's point isn't that one approach is wrong — it's that for program verification, where the extracted code is the product, having options matters. His own work uses Crane (C++ extraction) to build playable WebAssembly games from verified Rocq.

- **#concept** — **Infrastructure as institutional memory**: The ecosystem list isn't just a flex. CompCert, Iris, VST, Fiat Crypto, and the rest represent *qualified* verification infrastructure — tools that have passed regulatory review (ANSSI criteria, aircraft qualification) or shipped in production (browsers, TLS). A Lean port of any one is possible; a Lean port that inherits the qualification history is not. The regulatory acceptance section makes this explicit: even a perfect weekend port doesn't carry the certification paperwork.

- **#pattern** — **AI as tool-agnostic force multiplier**: Korkut's counter to the "AI writes better Lean" argument is the most broadly applicable insight in the post. If AI can generate proofs in either language, the bottleneck isn't the language the AI knows best — it's the language that has the ecosystem you need. This inverts the 2024-era logic that favored popular languages for AI assistance and aligns with the observation that [[We Have Proof Automation Now]] makes about proof irrelevance: if proofs are cheap, the *types* and *infrastructure* are what matter.

---

## Critical analysis

**The post is right about the facts, but the facts have a shelf life.** Korkut is scrupulously fair: he links to Lean alternatives for every Rocq feature he discusses, acknowledges Iris-Lean's rapid development, and provides runnable counterexamples with pinned toolchain versions. This is not a hit piece. But it's a snapshot, and every snapshot is vulnerable to the next Lean release. The coinduction gap is real in July 2026; if Lean 4.33 adds kernel codata, the strongest section of the argument evaporates.

**The ecosystem argument is the durable one, and it cuts both ways.** Rocq's ecosystem advantage is real and measured in person-centuries. But ecosystems are path-dependent, not merit-based. The fact that CompCert is written in Rocq doesn't mean Rocq is *better* for writing verified compilers — it means INRIA picked Rocq in 2005 and nobody has been willing to fund a Lean rewrite. The ANSSI certification is similarly path-dependent: it certifies a toolchain, not a language. A Lean-based verified compiler could in principle qualify; it just hasn't been through the process. Korkut is right that this matters *now*, but wrong to present it as a permanent property of the languages.

**The coinduction section is both the strongest and the most vulnerable.** It's the strongest because it's the most concrete: runnable examples, pinned versions, checked failures. It's the most vulnerable because it's about a missing feature, and missing features get added. QPFTypes is explicitly a proof of concept; Lean FRO employs people working on this; the gap could close in a single release. The nested inductive gap is smaller and more likely to persist — both languages are conservative about what their kernels accept, and changing these checks is genuinely hard.

**The AI section is the most interesting and least developed.** Korkut dismisses the "AI prefers Lean" argument in two paragraphs, but he's right to dismiss it. The deeper question — which he doesn't explore — is whether AI changes the economics of the ecosystem argument itself. If AI can port a library from Rocq to Lean in a weekend, does the ecosystem advantage shrink? If AI can generate proofs in either language equally well, does the coinduction gap matter less because you can generate the workaround? The post's implicit answer is "no, because the *qualified* infrastructure doesn't port," but this is a claim that deserves more scrutiny.

**What the post doesn't say is as important as what it does.** Korkut never claims Rocq is a *better language* — only that it's a better *fit for his work*. The distinction matters because "his work" is unusually demanding: interaction trees, game trees, verified games extracted to C++ with Crane. Most program-verification users don't need coinductive types or multiple extraction backends. For them, Lean's mathlib, active community, and AI momentum might genuinely be more valuable than Rocq's ecosystem. The post is a defense of a specific research program, not a universal recommendation — and it's honest about that, if you read the caveats.

**The regulatory acceptance section is a grenade thrown politely.** ANSSI certification and CompCert's aircraft qualification aren't academic concerns — they're "your product can't ship without this" concerns. Korkut buries this at the end, almost as an afterthought, but for anyone building verified software in regulated industries, it's the whole argument. A tool that has passed regulatory review is a different category of thing from a tool that hasn't, and the gap isn't closed by better type theory.

---

## Related pages

- [[We Have Proof Automation Now]] — Adam Langley's complementary argument from the other side: Lean + LLM-generated proofs as practical engineering. Where Korkut argues from ecosystem depth, Langley argues from proof automation cost collapse
- [[The Coming Need for Formal Specification]] — Congdon's case that AI code generation inverts the economics toward formal methods; Korkut's post is a practitioner's answer to "which formal method?"
- [[Lean Software Scaling Laws]] — Gwern's hypothesis that formally-strong languages like Lean have better scaling exponents for LLM predictability; Korkut's ecosystem argument is the counterweight: scaling laws matter less than accumulated verification infrastructure
- [[Software Engineering Craft]] — The hub page for fundamentals; formal verification is one of those fundamentals that doesn't change even when everything around it does
- [[Specifications as the Product]] — The durable artifact isn't the code or the proof; it's the specification. Korkut's work exemplifies this: the Rocq source is the spec, extraction produces the code

---
*Sources: [[raw/why-rocq-is-better]]*
*Last updated: 2026-08-01*
