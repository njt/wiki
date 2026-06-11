# TRIZ

A Soviet-born systematic innovation methodology that turns invention from trial-and-error into a structured lookup process. Genrich Altshuller analyzed hundreds of thousands of patents to discover that inventive problems and their solutions follow repeatable patterns — then built a toolkit (40 principles, contradiction matrix, IFR, ARIZ) around that insight. The method works: documented ROI at Intel ($212.5M over 21 months) and Samsung (50 patents in a single year) proves it. But TRIZ's enduring value isn't in its specific matrices — it's in the meta-insight that **innovation has structure you can learn**, and that most "creative breakthroughs" are just analogical transfers from other domains where the problem was already solved.

---

## Key Quotes

> "Problems and solutions are repeated across industries and sciences."

This is the entire thesis in one sentence. It's also the most underappreciated idea in innovation culture. We fetishize the lone genius having an original thought when the data says they're mostly pattern-matching against solutions from adjacent fields. The parallel in software: how many "novel" architectures are just rediscovered database techniques from the 1970s?

> "Patterns of technical evolution are replicated in industries and sciences."

Altshuller's laws of technical systems evolution are essentially a theory of technological convergence — the idea that systems evolve through predictable stages regardless of domain. If true (and the patent data suggests it is), this has real predictive power. You should be able to look at any immature technology and forecast its trajectory by finding analogous mature systems. TRIZ practitioners claim exactly this.

> Altshuller recognized that improving one parameter "often leads to the deterioration of another, a situation he termed a *technical contradiction*."

The contradiction is the unit of work in TRIZ. Instead of optimizing one parameter and accepting the trade-off (the normal engineering approach), TRIZ demands you resolve the contradiction entirely — find a solution where both parameters improve. This is what separates TRIZ from conventional design optimization. The IFR (Ideal Final Result) is the logical extreme: the function happens, with no mechanism. It sounds absurd, but it's a forcing function that prevents premature compromise.

> "The innovations have scientific effects outside the field in which they were developed."

This is the third core finding and it's the most actionable for practitioners. Your hard problem was probably solved in 1987 by a chemical engineer who never published in your field. TRIZ is essentially a **cross-domain solution transfer protocol** — the contradiction matrix is a routing table that says "for this type of conflict, look in *that* domain."

---

## Key Themes

- #concept — TRIZ is a meta-framework, not a tool. It's a theory of *how* invention works, from which specific tools are derived.
- #tool — The contradiction matrix is a structured lookup table. It doesn't require creativity — it requires pattern recognition and analogical transfer.
- #pattern — The 40 principles are design patterns for physical systems, published decades before software design patterns were formalized. Segmentation, asymmetry, merging, nesting, dynamism — these map directly to software architecture decisions.
- #person — Genrich Altshuller is one of the great overlooked figures of 20th-century engineering. Soviet inventor, Gulag survivor, systematic thinker who built a rigorous theory of creativity while the West was still treating brainstorming as the state of the art.

---

## Critical Analysis

**The matrix is probably wrong, and it doesn't matter.** The specific principle-to-contradiction mappings were derived from mid-20th-century Soviet patents. Mechanical engineering bias. But the *process* of using the matrix — abstracting your problem into a contradiction, then systematically scanning for analogous solutions — is where the value lives. The matrix is training wheels for a cognitive skill. Once you've internalized the pattern, you don't need the matrix anymore.

**TRIZ exposes how shallow most "innovation" consulting is.** Compare TRIZ to design thinking, brainstorming, or lateral thinking exercises. Those are vibe-based — they create conditions where innovation *might* happen. TRIZ is algorithmic — it gives you a procedure that, followed correctly, produces candidate solutions. The fact that corporate innovation programs overwhelmingly chose the vibe-based approaches over the algorithmic one tells you something about what companies actually want from innovation programs (hint: it's not solutions, it's the feeling of being innovative).

**The Gulag story matters.** Altshuller developed TRIZ's foundations while imprisoned for criticizing Stalin. He returned and kept building it. This isn't just biographical color — it's evidence that TRIZ was built by someone who had skin in the game. He wasn't an academic theorizing about creativity from a tenured position. He was an engineer who needed better ways to solve real problems, and the Soviet system tried to kill him for it. The methodology carries that pragmatism.

**The modern variants are a mess.** TOP-TRIZ, I-TRIZ, Liberating Structures' TRIZ — they're incompatible forks maintained by competing practitioners, each claiming to be the true heir. Classic "standards war" dynamics in a niche too small for anyone to win. MATRIZ (the International TRIZ Association) doesn't recognize the most actively developed variant as definitive. This is why TRIZ hasn't had its "moment" in the West despite proven ROI — there's no clean on-ramp, just a fractured landscape of practitioners selling their preferred flavor.

**TRIZ and AI agents are a natural fit.** The contradiction matrix is essentially a retrieval system — "given these conflicting parameters, what solutions exist?" — and LLMs are extremely good at analogical retrieval across domains. A TRIZ-aware agent could: (1) abstract a problem into its contradictions, (2) search across patent databases and scientific literature for analogous resolved contradictions, (3) propose adapted solutions. This is probably more useful than trying to teach an LLM to "be creative." The structure is already there.

---

## Related Pages

- [[Not-Knowing (Vaughn Tan)]] — Tan's uncertainty taxonomy and TRIZ share the same premise: diagnosis before action. Misdiagnosing the type of problem produces worse results than having no framework.
- [[The Mundanity of Excellence]] — Excellence as qualitatively different choices, not quantitatively more effort. TRIZ makes this operational: good solutions come from asking different questions (what's the contradiction?), not from trying harder.
- [[Things You're Allowed to Do]] — TRIZ's IFR is the ultimate expression of this idea. The constraint you think is real probably isn't. The Ideal Final Result says: assume the function happens by itself, then work backward to what's actually necessary.
- [[Load-Bearing Assumptions]] — The contradiction matrix is a load-bearing assumption finder. It forces you to name which parameters are in conflict, which surfaces hidden assumptions about what must be traded off.
- [[Creative Firewall]] — Sundar's framework for authentic vs. AI-optimized creativity. TRIZ predates this debate by 75 years and takes the opposite position: creativity IS structured, and pretending otherwise is romantic nonsense.
- [[Duck, Duck, Duck! (IDEO)]] — IDEO represents the vibe-based innovation culture that TRIZ was designed to replace. The duck is charming; the contradiction matrix works.
- [[Guardrails and Feedback Loops]] — "Linters beat prompts" is the software equivalent of "the contradiction matrix beats brainstorming." Deterministic enforcement over hope.
- [[First Principles]] — TRIZ pushes past first principles into a harder claim: not just "reduce to fundamentals," but "the fundamentals follow laws, and those laws are the same across domains."

---

*Sources: [[raw/wikipedia-triz]] — Wikipedia, https://en.wikipedia.org/wiki/TRIZ*
*Last updated: 2026-06-11*
