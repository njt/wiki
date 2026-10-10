# Four Provers, One Theorem (blueberrywren)

blueberrywren formalizes Euclid's proof of the infinitude of primes in Lean, Isabelle/HOL, HOL4, and Agda, then compares the four on user experience — automation, transparency, theorem search, interactivity — rather than on expressive power. The result is a rare like-for-like field report on formal methods tooling, with an honest declared bias toward LCF-style systems.

---

The experiment is deliberately minimal and therefore sharp: one classical proof, four systems, same structure everywhere — so all differences that show up are differences in *tooling and interaction*, not in what can be expressed. The two camps split on foundations: Lean and Agda carry proof terms via Curry-Howard; Isabelle/HOL and HOL4 reduce everything to a small proof kernel and embrace classical logic wholesale.

## Key Quotes

> "I'm more a fan of LCF because it seems more amenable to automation, and I don't see the point of carrying around proof terms (classical logic is too useful!)."

The thesis of the whole piece, stated upfront. Proof terms are the price dependent types pay for constructivity — and the author thinks classical logic buys more than executability does.

> "Sometimes `grind` is smarter than `sledgehammer` and sometimes it's stupider than `simp`. Weird."

The sharpest observation about Lean's automation: it's powerful but bimodal and inscrutable. Notably, both `sledgehammer` and `grind` are all-or-nothing, while partial tools (`simp`, `auto`, `gvs`) "make some progress and then leave the context in a better place" — which the author argues is what actually matters day to day.

> "Very often you know the *shape* of something you want, but maybe not the lemma itself."

On why Isabelle/HOL and HOL4's pattern-based theorem search (`find_theorems`, `DB.match`) is such a killer feature — and why Lean's leansearch/loogle and Agda's nothing fall short. Search shapes the experience more than the proof language does.

> "The disadvantage is that it is *horribly* slow to do so... Is it worth giving up the LEM and good proof automation? You decide."

Constructivity's dividend, priced honestly: the Agda proof is executable and can emit an actual prime larger than n — after checking primality around 360,000 inside the factorial construction. Programs-from-proofs are real but rarely free.

> "The Lean proof was... such a pain. I had to put more effort into that one than any of the others, despite Lean having reasonably decent automation!"

A useful data point against reputation: the most-hyped system required the most manual fiddling on a small proof, mostly from lemma-hunting friction.

## Key Themes

#formal-verification #concept #tool #comparison #automation

## Analysis

The honest contribution here is methodological: by fixing the theorem and the proof, the piece isolates user experience as the variable, which is exactly the axis most tool comparisons hand-wave. The rankings are confessedly arbitrary, but the underlying observations are concrete and transferable — automation that makes *partial* progress beats automation that is occasionally brilliant but all-or-nothing; searchable theorem databases by pattern beat name-based lookup; being able to see a proof mid-construction changes how you work.

It's also a quietly useful corrective to hype in either direction. HOL4 — the least fashionable of the four — wins the overall ranking on the strength of its REPL interaction model ("HOL4 proofs are literally just SML terms!") and its search tooling, while Lean, the fashionable one, comes with warnings about unpredictable automation and painful lemma discovery. And the Agda segment is the most philosophically interesting: it's the only system where the proof is a *program*, and the author prices that dividend against the cost of writing an entire primality decision procedure by hand.

The limitation is the sample size of one proof and one author, by their own admission ("Don't expect an unbiased review"). Euclid's proof is small and arithmetic; conclusions about automation may not transfer to heavy algebraic or concurrent formalization. Still, as a UX comparison at fixed workload it's more rigorous than most tool writing.

## Relations

This source gives concrete, first-hand texture to [[The Coming Need for Formal Specification]]'s abstract argument that AI-era engineering will push verification upstream — its whole point is what the *experience* of using verification tools is actually like, and its finding that partial-progress automation beats all-or-nothing solvers complicates the assumption that LLMs will simply generate proofs for these systems. It also nuances [[The GUS Stack — Go, Unix, SQLite]]'s training-data-density thesis: Lean is the training-data-dense, fashionable tool, yet the author had the *most* trouble with it on a small proof — tool familiarity and interaction model matter as much as ecosystem mass. And it sits naturally alongside the craft-tradition pages under [[Software Engineering Craft]], showing that proof engineering has its own versions of the simplicity-versus-magic tension the rest of the wiki documents.

---
*Sources: [[raw/primes]], [[summary/primes]]*
*Last updated: 2026-10-10*
