# IDRs as Records of Intent

Will Lachance's field correction on his own IDR (Implementation Decision Record) practice: an LLM asked to write a decision record from a vague prompt will fill it with a textual description of what it planned to implement anyway, producing a lossy English restatement of the diff. An effective IDR must keep returning to **intent** — the itch, the goals, the non-goals — or it is worth less than the diff it describes.

---

Lachance invented IDRs (a pun on ADRs) for two purposes: better LLM output, and a durable record of decisions made during implementation. The post is a correction to his own earlier framing. The failure mode he names is important: the agent doesn't refuse to write the record — it complies, and the compliance is the problem. A vague prompt yields a verbose, content-free document that *looks* like governance.

The worked pair of examples does the teaching. The bad IDR (DuckDB serialization for repo-parser) has a problem statement that "isn't horrible" but is too vague to give the agent a goal — and an "almost inscrutable LLM context" section. The result was "ok-ish", he doubts the IDR added value, and he cringes re-reading it. The good IDR (PyPI publishing via GitHub Actions) is a few short sections: the itch, goals (easier releases, security), an explicit non-goal (no automated version bumping), and a proposed solution pointing at the official packaging guide and a known-good workflow. "Every sentence does the work of steering behaviour towards the actual result I want to see, rather than being a set of instructions that it might misinterpret."

---

## Key quotes

> left to their own devices given a vague prompt, an LLM will often just try to fill one in with a textual description of what it planned to implement anyway. This makes them pretty lossy as a record of intent (it's basically not much better than a diff, and arguably worse)

The LLM-compliance trap: agents write plausible-looking decision records that record nothing. This is the same shape as the problem [[AI-Written Change Descriptions]] attacks — generated prose that describes the surface while omitting the framing a future reader needs.

> for an IDR to be really effective it needs to keep returning to the **intent** and decisions taken to support it. Without this, it's just a long-winded way of restating the diff in english.

The whole test in one sentence. Intent — the itch, goals, non-goals — is the part the diff cannot carry; everything else is duplicate.

> Bonus: when using an LLM it provides clearer guard-rails. Every sentence does the work of steering behaviour towards the actual result I want to see, rather than being a set of instructions that it might misinterpret.

Short intent-dense specs outperform detailed instruction lists as steering: fewer degrees of freedom to misread. This connects to [[Removing the Noise for Better Prompts]] — every sentence must earn its place.

> If I need more detail, I'll just look at the diff.

The division of labour: the record holds why, the diff holds what. Anything the record says that the diff also says is waste.

## Key themes

- #concept — intent as the only non-redundant content of a decision record
- #pattern — terse goals + explicit non-goals as steering guardrails for agents
- #tool — IDRs, ADR-adjacent markdown artifacts in the repo

## Analysis

The value of this post is its honesty about a practice's failure mode. Most spec-driven-development writing argues for *more* documentation; Lachance argues for *less, aimed better* — and shows that the agent itself is a unreliable author of the artifact. Two sharp implications:

First, the bad example indicts a whole class of "the agent wrote the spec" workflows. If the spec is generated from the same vague prompt that generates the code, it adds nothing — it's the diff paraphrased, with a title. Spec-first only pays when the human supplies the intent the agent cannot invent. His good IDR is almost entirely *his* content: the itch, the non-goals, the pointer to best practice.

Second, the "arguably worse than a diff" claim deserves pushback only mildly — a bad IDR is worse than a diff because it *pretends* to carry decision information, adding a false sense of understanding. That makes poorly-written records an active liability, the same dynamic as silently-resolved ambiguity: the artifact exists, so nobody asks the question it was supposed to answer.

## Related pages

This sharpens [[Coding Agents Love Decision Records]] from the other direction: Duncan Davidson worries about agents over-adhering to stale ADRs; Lachance shows the upstream failure — agents authoring ADRs that never contained a decision in the first place. It complements [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]]: a vague-prompt IDR is exactly the unsigned debt entry — a record exists, but no intent was ever recorded. And it reinforces [[Capturing Why Engineering Decisions]]: the durable artifact must carry the why, because the diff already carries the what.

---
*Sources: [[raw/idrs-as-records-of-intent]], [[summary/idrs-as-records-of-intent]]*
*Last updated: 2026-10-14*
