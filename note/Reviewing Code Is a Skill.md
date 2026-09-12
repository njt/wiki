# Reviewing Code Is a Skill

An unnamed practitioner's essay arguing that code review is a trainable human craft, not a fixed-cost bottleneck waiting to be automated away. Against the 2025–2026 memes declaring review dead, slow, or already superseded by LLMs, the author marshals three bugs he caught that high-end LLM reviewers missed, the code-review research literature, and a set of concrete experiments for getting better — all to make one claim: if humans keep shipping code, reviewing it well is one of the highest-leverage skills they can develop.

---

## Key Quotes

> "It's possible to get better at reviewing code... it's possible to teach someone to get better at reviewing code... we don't quite know where the human skill ceiling lies."

The thesis in three propositions. The middle one is the radical part in an AI-heavy moment: review is not an innate gift you either have or don't, but a *teachable* skill. The third quietly undercuts the "LLMs will leave humans in the dust" fatalism — if we don't know the human ceiling, "humans lose by default" is an assertion, not a measurement.

> "The other thing to note is that LLM reviews with a mixture of high-end coding models (around Jun 2026) were run for all of the PRs described below. They did not catch the issues that I caught."

A single practitioner's anecdote, not a study — but it lands precisely on the weakest joint in [[The End of Code Review]]'s argument, which asserts the "set of defects that escape agent review but are caught by human review is shrinking." Here are three that escaped *and* were caught, in ordinary infrastructure work, by a human who'd trained the skill. The plural of anecdote isn't data, but it's the counterexample the synthesis paper hand-waves past.

> "I tend to think of programs in terms of invariants and little proofs."

The author's self-diagnosis of *why* he catches bugs others miss — and his pick for the most "learnable and teachable" ingredient. The checksum outage is the clearest illustration: S3 gives all-or-nothing writes to a single object but no multi-object transactions, so "tarball, then checksum" to fixed names is an invariant that can be violated by a crash between the two writes. Thinking in invariants surfaces the failure mode before the code runs.

> "If you're able to drop down a lower-level of abstraction compared to your peers, that's almost always an advantage."

His answer to "why invest in review when other skills have higher leverage?" Understanding the layer beneath — SQL query plans, memory allocation, the `aws` CLI's version history — widens the set of problems you can solve and the failure modes you can see. This is the same move as [[The Joy and Power of Understanding]]: force before multiplier.

> "What is the most praiseworthy thing of all? / Skill."

The closing Yaksha–Yudhishtira exchange from the Mahabharata, quoted as the epigraph-in-reverse. After several thousand words of argument, the answer is a one-word koan: skill — acquired, practiced, teachable — not innate brilliance or tool access.

## Key Themes

- #concept **Reviewing code as a skill** — learnable, teachable, with an unknown human ceiling. The post's whole thesis, and its point of departure from both the "review is dead" and "review can't be fixed" camps.
- #concept **Review's four (plus) purposes** — education, norms, gatekeeping, accident prevention (Google); plus knowledge transfer, team awareness, and understanding (Bacchelli & Bird). The author uses this to refuse single-purpose framings like "code review is for X, not Y."
- #pattern **Invariants and little proofs** — the transferable mental move behind catching all three bugs, and the piece's candidate for "the thing you can teach."
- #comparison **Human vs. LLM review** — the author runs both and reports the LLMs missed what he caught, while conceding the premise that LLMs may improve faster.
- #pattern **Training experiments** — Socratic review dialogues, near-miss post-mortems, firewalled modeling (formal models built without reading the code), and expertise extraction via Applied Cognitive Task Analysis.

## Critical Analysis

**The strongest thing here is the refusal to be sloppy about scope.** The author is careful that "valuable to get better at reviewing code" is *conditional* on believing humans will remain in the loop — and explicitly separates "is the premise true?" from "does the implication hold?" That's more intellectual honesty than most of the "code review is dead" takes he's pushing back on, which smuggle the premise in as settled.

**The three bugs are the payload.** Everything else is scaffolding. What makes them land is their ordinariness: a file lock, a CLI flag's version, a checksum ordering. None required genius — each required *attention to the invariants underneath*, which is precisely what a trained reviewer develops and a mix of frontier models, run on the same PRs, missed. This is the concrete counterweight to [[Agentic Code Review]]'s "review harder is a fantasy": review *better* isn't a fantasy, it's a trainable practice — even if it can't close the volume gap alone.

**The "mad scientist" section is underdeveloped but generative.** Each idea is a seed, not a recipe — the author admits he hasn't tried them. The Socratic-dialogue one is the most promising: it targets the *thought process* rather than the diff, the exact axis [[AI-Written Change Descriptions]] identifies as missing when reviews descend on code without the higher-level framing. The firewalled-modeling idea dovetails with [[Ways of Checking]]'s core move — check differently, don't re-scan the same surface — since a model built *without* reading the code is a genuinely different instrument from the code itself.

**Where it's thin.** The author declines to engage the bottleneck math that [[The End of Code Review]] makes central: even if review is a learnable skill, human review *capacity* doesn't scale with agent throughput. "Get better at review" and "you can't review everything agents produce" are both true, and he only defends the first. The essay would be stronger if it named what it's *not* claiming — that skill development substitutes for pipeline restructuring. It doesn't; the two arguments are complementary, not competing. The post also complements [[Understand to Participate]]'s participation thesis from the craft side, but stops short of Litt's framing that the loss is *agency*, not safety.

**The honest gap: no author, no date.** The post is unsigned and undated on typesanitizer.com. The biographical detail (physics PhD dropout, 2019) is offered as evidence of "I learned this, I wasn't born with it," but there's no way to verify the claimed review record or the LLM comparison. Read it as a practitioner's field notes with a strong thesis, not as data.

---

*Sources: [[raw/code-review-html]], [[summary/code-review-html]]*
*Last updated: 2026-08-14*
