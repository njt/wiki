# Bend — A Proof-Gated Language for AI-Written Code

Bend (bend-lang.com) is a new programming language whose design center is a reversal: the programmer is an AI agent and the human is a legislator. Humans write `LAWS.bend` — precise, machine-checkable statements of what must never break; the AI writes the code *and* `PROOF.bend`, proving every law; and a proof checker engineered to finish in under a second gates every commit, so the agent verifies its own work after every change. Python-shaped syntax, affine types, C-class native speed and CUDA-class parallelism come along as the carrier. The one-line thesis: "`LAWS.bend` is `AGENTS.md` backed by proof."

---

## Key Quotes

> "a **fast** language that **blocks AI mistakes** via **proof** — **C** speed · **CUDA** parallelism · **Lean** proofs · **Python** syntax"

The tagline is the whole positioning: Bend is not selling productivity, it is selling *prevention*. Compare the wiki's accumulated evidence that agents ignore prose instructions — the gap [[A Convention Is Not a Constraint]] names. Bend's answer is to make the constraint non-prose.

> "In the post-AGI economy, humans will eventually stop writing and reading code, but we still need an ambiguity-free way to tell the AIs building the world around us what we want done. With **laws**, our intents can be much more precise than natural language."

This is [[The Coming Need for Formal Specification]]'s argument taken to its literal endpoint — not "consider formal methods" but "we built the language where the spec is the durable artifact and code is the disposable part."

> "How can you **trust** code you never read? By demanding a **proof**."

The cleanest statement of the review-bottleneck problem anywhere in this wiki's source corpus. Review doesn't scale with generation; proofs do, because checking is mechanical and (Bend claims) sub-second.

> "Without LAWS.bend, the bug went live. With LAWS.bend, the AI had to retry until it built a wall and proved the law holds. Merging a bug is mathematically impossible: it is a *theorem*."

The demo is a feedback loop, not a filter: the agent retries against the checker until the proof lands. That is [[Guardrails and Feedback Loops]]' "loop that makes quality self-correcting" implemented in the type system — the same shape as [[Lean Software Scaling Laws]]' prediction that formally-strong languages scale better with LLMs, but productized.

> "`LAWS.bend` is `AGENTS.md` backed by **proof**. 'Make no mistakes' is now *type-checked*."

The sharpest one-liner in the steering-file space. Anthropic's taxonomy in [[Steering Claude Code]] treats instruction files as advice that loads into context; Bend turns the steering file into a gate the agent cannot talk its way past.

> "Laws are a critical feature in Bend, as they provide an ambiguity-free language on which humans can state precise specs for AI's to implement... We envision that 'law-driven development' will eventually become the way humans use AI to write and maintain large codebases, as it is the perfect middle point between having to code everything manually (laborious) and letting AI do it all via prompts without auditing a line of code (error/ambiguity-prone, unsecure)."

The guide's own term for the practice is "law-driven development." Note the division of labor it hard-codes: the human writes the claim, the AI writes everything else — the code *and* the proof.

> "Bend does almost no inference, meaning it requires more annotations than similar languages. This is what allows Bend's checker to be significantly faster than other provers."

The load-bearing trade of the whole project. Verbose-for-humans, fast-for-checkers is normally a bad deal; it becomes a great deal when the "human" is a model that doesn't mind verbosity and the checker sits inside the agent's inner loop.

---

## Key Themes

#tool #concept #formal-verification #spec-driven #guardrails

---

## Critical Analysis

**What is genuinely new here is not proofs — it is where the checker sits.** Proof assistants meeting LLMs is already documented in this wiki: [[We Have Proof Automation Now]] has an LLM building a Zstandard decompressor in Lean. Bend's contribution is an engineering constraint turned into a language design: if the agent should check after *every change*, the checker must be sub-second, and every other decision — almost no inference, mandatory annotations, affine-only variables, mandatory termination, no mutual recursion, no tactics — is sacrificed on that altar. That is a coherent design philosophy, and it inverts forty years of language-design defaults aimed at human convenience. The insight worth keeping even if Bend fails: **when the primary reader of code is a model, verbosity is free and checker latency is the scarce resource.**

**The "AGENTS.md backed by proof" framing resolves a gap this wiki keeps documenting.** [[Architectural Guardrails for AI-Generated Code]] shows agents bypassing banned code paths because the ADR was prose no one fed them; [[PAAD — Defense-in-Depth for AI-Assisted Development]] bolts verification skills onto Claude Code from the outside. Bend moves the enforcement *into the language*: the law is not context the agent might read, it is a gate the agent must satisfy to ship. "Parse, don't validate" ([[Parse Don't Validate]]) taken to its terminal form — the illegal state is not just unrepresentable, it is uncommittable.

**But "merging a bug is mathematically impossible" is overclaimed, and the fine print admits it.** It is a theorem *relative to the laws you wrote*. The demo law — "no move sequence leads to victory" — is a clean global property. Most real bugs are not: the wrong UI text, a misread business rule, a security hole outside any stated law. Proof coverage is correctness-as-spec'd, and the gap between the law you wrote and the law you meant is exactly where [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] locates the debt — a proof can ratify a misunderstanding with mathematical confidence. The escape hatches compound this: `@unsafe` defs "fall outside Bend's proof guarantees", and "Under the Hood" concedes the theory admits `Type : Type` and no positivity check, held together by a live/dead "wall between two checking modes", with the Lean mechanization (`bend.lean`) lagging the implementation (`bend.ts`). "Unquestionable mathematical correctness" is marketing resting on a checker whose own consistency story trails its code.

**The labor question is unaddressed: who writes the laws?** The pitch says the human writes `LAWS.bend` and the AI never touches it. If stating a property precisely is hard — and Hillel Wayne's schoolbus quip about TLA+ experts, cited in [[The Coming Need for Formal Specification]], says it is — the bottleneck has not been eliminated, it has moved from reviewing code to legislating properties. Bend's bet is that this is the *right* place for the bottleneck, which is defensible; but "law-driven development" will live or die on whether ordinary developers can state properties worth proving, not on whether models can prove them. There is also an empirical tension worth flagging: [[Constraint Decay]] found agents *lose* ~30pp pass rate when structural constraints are imposed, while Bend bets that hard constraints make agents *better* — the resolution is probably that a sub-second, perfectly-scoped checker is a different animal from an ORM's implicit conventions, but nobody has measured that yet.

**The parallelism story is real but orthogonal.** Purity plus affinity gives lock-free fork-join parallelism and a single C file that compiles as both CPU program and GPU kernel — genuinely elegant, and the strongest *conventional* language-design contribution on the page. But most agent-written code is IO-bound CRUD where the GPU pitch is irrelevant; the parallelism is the carrier for the proof story, not the story itself.

**Bottom line:** the first language whose design center is "the programmer is an agent." Its bet — that the durable artifact is a machine-checkable law and everything downstream is disposable — is the most committed version of the spec-as-product thesis in this wiki. Its risk is equally concentrated: if humans can't legislate properties at scale, `LAWS.bend` becomes a demo artifact and Bend is a fast pure language with an unusual hobby.

---

## Cross-Links

- [[We Have Proof Automation Now]] — strengthens it: Langley showed LLM-generated proofs make dependent types practical in Lean; Bend productizes the same bet at the language level, adding the twist that the checker must run in the agent's inner loop, not in CI.
- [[The Coming Need for Formal Specification]] — this is the tool Congdon's essay calls for, and also a nuance: Bend buys checker speed by deleting inference and tactics, i.e. by making verbosity the price — the cost Congdon's argument waves away.
- [[Steering Claude Code]] — complicates its taxonomy of instruction-delivery mechanisms: where CLAUDE.md and rules load *advice* into context, `LAWS.bend` is the steering file promoted to an enforceable gate — the site itself bills it as "AGENTS.md backed by proof".
- [[Lean Software Scaling Laws]] — a designed-to-the-prediction artifact: Gwern proposed that formally-strong languages have better LLM scaling exponents, and Bend is a language built explicitly around what a model can prove — the hypothesis turned into a product before the measurement ran.

---

*Sources: [[raw/bend-lang-com]], [[summary/bend-lang-com]]*
*Last updated: 2026-09-19*
