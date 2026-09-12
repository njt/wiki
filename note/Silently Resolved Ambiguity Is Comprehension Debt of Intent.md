# Silently Resolved Ambiguity Is Comprehension Debt of Intent

A short post that splits "comprehension debt" into two kinds and argues the senior kind is the one nobody talks about: when an agent silently resolves an underdetermined requirement, nothing records that a decision was made at all — a debt entry with no signature. The fix is not to understand every change, but to decide which decisions are yours and engineer a tripwire that surfaces exactly those.

---

## Key Quotes

> "Agents resolve ambiguity constantly. An issue underdetermines a behavior, the agent picks something statistically probable, and nothing surfaces the fact that a decision was made at all. That's a debt entry with no signature: comprehension debt of *intent*, senior to comprehension debt of implementation."

This is the thesis, and it's sharp. Comprehension debt of implementation is "the code does something I don't understand." Comprehension debt of intent is "the code does something because nobody — including the agent — noticed there was a choice to make." The second is harder to audit precisely because the moment of decision left no artifact. It compounds silently under every PR.

> "Someone who only vibe codes will get shockingly far, and then discover they've made something that cannot be changed while maintaining trajectory. Someone who understands every change will be using spicy autocomplete, and be too slow. Something is in the middle, we posit – intentional decision points backed by data, and not getting involved much past those points unless something seems off the rails."

The spectrum frame is more useful than the usual "vibe coding bad / review everything good" binary. Both poles are failure modes; the productive stance is *selective* attention, and the scarcest resource is knowing where to spend it. This is the same conclusion [[Optimizing for Decision Points]] reaches from the workflow-design direction.

> "Decisions made with awareness reduce comprehension debt but incur naps; decisions you allow (intentionally or accidently) your agent to make prevent the nap but increase debt."

The "nap" is the honest word everyone dances around. Attention is expensive, and the whole point of delegation is to buy it back. The insight is that this is a *trade*, not a sin — the failure mode isn't delegating, it's delegating without noticing which decisions you're giving away. The nap/debt exchange rate is the real design surface.

> "What's the tripwire that says 'stop, this one goes to a human,' without routing *everything* to the human?"

The core engineering question, stated as a design problem rather than a philosophy problem. The author refuses the lazy answer (escalate everything) because that defeats the nap savings. This is exactly the tension [[Optimizing for Decision Points]] frames as "meta-judgment" — the agent's job is not to have taste but to know when taste is required.

> "What does an outcome tie back to? Something I defined, a reference, or probability and LLM?"

A three-way provenance classification. "Something I defined" = authored intent; "a reference" = verifiable grounding; "probability and LLM" = a statistically probable guess, which is either usefully creative or an inaccurate hallucination. The author credits Ethan Zuckerman as beginning to take on provenance, and argues knowing a decision's provenance lets you improve the model through instruction rather than just patching around it.

## Key Themes

#concept #pattern #person

- **#concept — Comprehension debt of intent**: The senior debt. Not "code I don't understand" but "a decision made with no author and no record." Sits above comprehension debt of implementation and above the plain technical debt it feeds.
- **#pattern — The tripwire**: A workflow primitive that routes *only* ambiguity-requiring-intent to the human, not everything. The author leaves it as an open problem — the interesting part is that he treats it as solvable with better surfacing, not more review.
- **#pattern — Decision points with provenance**: The middle path between vibe coding and spicy autocomplete. Decide which decisions are yours; make the system surface them with data; let provenance tell you which kind of decision you're looking at.
- **#person — Ethan Zuckerman**: Named as the person beginning to take on provenance — the mechanism for auditing where an outcome came from.

## Critical Analysis

**The one-sentence contribution is real.** The field already has [[Cognitive Debt]] ("code cheaper to produce than to perceive") and Storey's intent debt from [[Five Studies That Are Changing How I Think About AI in Software Engineering]] — but those describe debt that *lives somewhere*: in people, in artifacts. Bl00cyb adds the temporal dimension: the debt that never gets created *as an entry at all*. A silently resolved ambiguity produces no "not knowing" because nobody knows there was anything to know. That's why it's senior — it's invisible to the audit trail that would otherwise catch the other debts.

**The "nap" framing is the unsung contribution.** Most writing on human-in-the-loop moralizes: you *should* stay engaged. Bl00cyb treats attention as a budget with an exchange rate — naps cost debt, awareness costs naps — which immediately reframes the problem from "how do I stay engaged" to "where do I spend engagement." [[Human-in-the-Loop is Tired]] diagnoses the exhaustion; this post gives you the accounting to decide what's worth staying awake for.

**The tripwire is gestured at, not built.** The author is explicit that the design problem — when does an ambiguity need authored intent — is *the* follow-up, and then pivots to provenance as the catch-all. That's honest, but it leaves the hardest work undone, and it's the same bootstrapping gap [[Optimizing for Decision Points]] flags: the framework risks requiring taste to apply, which is the thing it was supposed to externalize. The piece reads as a mid-series hypothesis post ("I'm writing up an experiment along these assumptions"), so this is a known hole rather than an oversight.

**Provenance as the load-bearing idea.** The three-way classification ("something I defined, a reference, or probability and LLM") is a genuinely workable schema, and it connects the post to a live thread — Zuckerman's provenance work, [[The Log is the Agent]]'s event-sourced lineage, [[ProofEditor]]'s tracked attribution. But the post doesn't grapple with the hard case: an agent's guess that *was* creative and turned out to be the right call. Provenance tells you it was "probability and LLM" — it doesn't tell you whether that was a bug or a feature. The distinction between "usefully creative" and "inaccurate hallucination" is only knowable in retrospect, which is a different kind of debt the classification can't resolve.

**Where it slots in.** This is the intent-debt half of Storey's taxonomy, operationalized as a *process* rather than an artifact problem. It complicates [[Loop Engineering]]'s "comprehension debt" warning by splitting it in two and saying the loop engineer's real exposure is the half the loop never logs. And it's the conceptual cousin of [[The Knowledge Chipper]] — that post laments the understanding the model built and threw away; this one laments the decision the model made and never told you it made.

---

*Sources: [[raw/silently-resolved-ambiguity-is-comprehension-debt-of-intent]], [[summary/silently-resolved-ambiguity-is-comprehension-debt-of-intent]]*
*Last updated: 2026-08-25*
