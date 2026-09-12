# An Introduction to Formal Logic (Peter Smith)

A 420-page textbook on classical first-order quantification theory, written for first-year philosophy undergraduates by a Cambridge logician, now freely available as a self-published PDF. The book builds from the most basic distinctions (validity vs. soundness, deduction vs. induction) through propositional logic to quantified logic with identity, using a Fitch-style natural deduction system. It is unusually rigorous about _why_ formal logic works — the metatheory is treated seriously, not deferred.

---

## What It Covers

The book has three major arcs, each prefaced by an "Interlude" that motivates the transition:

1. **Informal logic (Chapters 1–7):** Validity, soundness, inference forms, the counterexample method, and the move from informal argument patterns to formal schemas. The key distinction: _validity_ is about whether truth of premises guarantees truth of conclusion; _soundness_ adds that the premises are actually true.

2. **Propositional logic — PL (Chapters 8–24):** Truth-functional connectives, syntax and semantics, truth tables, tautological entailment, expressive adequacy, and a Fitch-style natural deduction proof system for PL. Includes metatheory: soundness and completeness of the PL proof system.

3. **Quantificational logic — QL (Chapters 25–42):** Names and predicates, quantifier-variable notation, formal QL languages, translations from English, and a natural deduction proof system for QL extended with identity (QL=). Covers definite descriptions and functions.

An appendix sketches the soundness and completeness proofs for QL.

## Pedagogical Approach

Smith takes a "slow-then-accelerate" approach — the first fifty pages are informal argument analysis before any logical symbols appear. He justifies this explicitly: students need to understand _what_ formal logic is for before they can appreciate _how_ it works.

The book's real strength is **metatheoretic honesty**. Unlike many introductory texts that treat the proof system as a black box, Smith names the gap between "these rules seem right" and "these rules are provably complete." Even if he only sketches the completeness proof, he makes clear that there _is_ a proof to be had, and why it matters.

The writing is conversational but precise. Smith acknowledges tradeoffs openly — the preface admits no text will please everyone and explicitly invites reader feedback for a planned third edition.

## Key Concepts

- **Deductive validity**: An inference is valid iff there is no possible situation where premises are true and conclusion false. The notion of "possible" here is the weakest, most inclusive sense — not physical possibility, but logical coherence.

- **Logical form**: Arguments share inferential patterns. Schemas (with recurring symbols indicating where substitutions must be uniform) are the tool for revealing those patterns. No argument has a _unique_ form — there is only the most general _reliable_ form it instantiates.

- **Truth-functionality**: The connectives of propositional logic are truth-functional — the truth-value of a compound sentence is determined entirely by the truth-values of its components. This is what makes mechanical decision procedures possible.

- **Expressive adequacy**: A set of connectives is expressively adequate if every truth-function can be expressed using only those connectives. The standard set {¬, ∧, ∨, →} is adequate; so are {¬, ∧} and {¬, ∨}. The Sheffer stroke (NAND) is adequate all by itself.

## Sharp Takes

**The "just teach them truth tables and move on" approach is intellectually dishonest.** Smith dedicates an entire chapter (Chapter 19: "If's and →'s") to the mismatch between the truth-functional conditional and the English "if/then", refusing to paper over it with hand-waving. This is the right call — students who learn the material conditional without understanding its limitations become the programmers who confuse logical implication with causal reasoning.

**The tree-vs.-deduction debate is a proxy war about pedagogical values.** The first edition used semantic tableaux (truth trees); the second switched to Fitch-style natural deduction after "many looking for a course text complained." Smith's ideal third edition would cover _both_, letting instructors choose. The real question: should students first learn to _find_ counterexamples (trees) or _construct_ proofs (deduction)? Smith's answer — both, eventually — is correct and almost nobody does it.

**The interlude chapters are the book's secret weapon.** Rather than lurch from topic to topic, Smith inserts short "Interlude" chapters that explain _why_ the next formal apparatus is needed. This meta-commentary — explaining the motivation before presenting the machinery — is what separates a textbook from a reference.

**Classical first-order logic is treated as a "permanent achievement," not a stepping stone to something fancier.** Smith is unapologetic about this. Non-classical logics, modal logic, many-valued logic — none appear. The bet is that mastering one system deeply beats surveying many superficially. For a first course, this is clearly right.

## Relevance to AI and Software

The connection is indirect but real. Formal logic is the intellectual ancestor of:
- Type systems and program verification (the Curry-Howard correspondence)
- SAT solvers and SMT (built on propositional and first-order logic)
- Database query languages (SQL is first-order logic in disguise)
- Model checking and formal specification
- The very notion of a "specification" as a logical formula a program must satisfy

Smith's book doesn't draw these connections — it's a philosophy textbook, not a CS one. But the discipline of distinguishing syntax from semantics, understanding what a formal proof actually _guarantees_, and reasoning about the limits of formal systems is directly relevant to anyone building systems that must be correct by construction. See [[We Have Proof Automation Now]] for the modern extension, [[Notes on Structured Programming]] for Dijkstra's application of logical reasoning to code, and [[Accordant]] for model-based testing as executable specifications.

---

*Sources: [[raw/introduction-to-formal-logic-smith]]*
*Last updated: 2026-07-29*
