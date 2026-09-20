# Making Your Data Ready for Agentic AI

This article argues that the entire discipline of data architecture was shaped for a consumer that no longer dominates: the human analyst, who supplies context, judgment, and skepticism for free. Agents supply none of it — and the claim that gives the piece its spine is that this gap must be closed by engineering, not by better models. Every implicit human behavior (pausing at a suspicious number, knowing what "revenue" means locally, being able to explain a decision afterward) has to be pushed into the data itself as one of five attributes: trusted, contextual, traceable, governed, operational.

---

## The argument in one paragraph

The article claims that when agents become the primary consumers of enterprise data, the binding constraint is no longer model capability but data readiness — and specifically that a data architecture built around "good enough for a human who'll double-check" is actively dangerous when handed to an agent that acts without hesitation. It claims this is fixable through four layered investments (contracts and quality, traceability and governance, a three-part context layer, tiered access up to write-back) plus observability woven through all of them from day one, and that skipping the lower layers is why most agentic AI programs stall. This is falsifiable: if agents with retrieval-augmented prompting and self-verification can compensate for untrusted, context-free data at acceptable cost, the whole layered stack is expensive over-engineering — and the article's own AtScale benchmark (under 20% to over 92.5% accuracy on the same model) is the strongest evidence it isn't.

## Key quotes

> A human hesitates at data that looks wrong; an agent acts on it anyway

The behavioral gap the whole article is built around, stated as a pull quote early and never really improved upon. Everything downstream — contracts, quarantine, confidence routing — is an attempt to give the architecture the hesitation the agent lacks.

> language models are gullible, they believe whatever they are handed and act on it

Borrowed from Simon Willison, and correctly deployed: it reframes data quality from an analytics nicety to a security-adjacent property. A gullible consumer doesn't degrade gracefully on bad input; it propagates it confidently.

> The semantic model doesn't make the agent smarter. It stops it from guessing.

The most quotable line in the piece, and the one that most cleanly separates this argument from "better models will fix it." Constraint, not intelligence, is the mechanism — which is why the definitions belong in version control rather than in prompts.

> Reversibility predicts safe autonomy better than the size of the transaction

A genuinely non-obvious design principle buried in the capability-model section: a $50,000 reversible ledger correction is safer to automate than a $200 irreversible external payment. It quietly contradicts the article's own staged-autonomy table, which keys guardrails to transaction size — the text notices this and says "prefer keying them to reversibility," but the table stands uncorrected.

> When agents become the primary consumers of your data, your data architecture *becomes* your AI architecture.

The thesis in one sentence, and a real inversion of how most organizations are sequencing this: they buy agent frameworks first and discover the data layer missing underneath. Whether it's fully true depends on how much agent-side context engineering can substitute for data-side structure.

## Critical analysis

The strongest move here is the framing of the five attributes as "the flip side of something a human used to do for free." That converts an abstract checklist into an accounting of labor that was never on anyone's balance sheet, and it makes the article's build order legible: you can't attach meaning to data you can't trust, and you can't safely let agents act on meaning you haven't declared. The freshness-SLA detail is the kind of practitioner texture that separates this from vendor content — key the SLA to last successful load rather than last value change, so steady data isn't flagged stale and a stalled pipeline can't masquerade as fresh. The same precision applied to vector indexes (the clock measures re-indexing against sources, not content change) catches the "silently failed indexer" case that most RAG deployments genuinely don't handle.

The three-model context layer — domain, semantic, capability — is the article's most original structural contribution. Splitting nouns, numbers, and verbs, and insisting they share one vocabulary ("A refund acts on the same customer the revenue figure counts"), is a real design constraint, not a taxonomy. The "retrieved text informs, it never gates" boundary is equally strong: it's simultaneously a governance rule and a prompt-injection defense, since a poisoned document can shape what the agent proposes but can never authorize what it does. The article is honest about the limits — injected text can still influence proposals and fool human approvers — which is more candor than most pieces at this altitude offer.

The weaknesses are the weaknesses of the survey genre. The article gestures at open problems and then moves past them: turning heterogeneous quality signals into a single confidence score is admitted to be unsolved ("start with a hard gate rather than a smooth composite"), which is honest but means the confidence-threshold routing section is thinner than its billing. The staged-autonomy ladder is borrowed intuition — the new-hire analogy does real work — but the evidence that promotion "turns on evidence, not a hunch" is deferred to a testing discipline explicitly declared out of scope. The survey statistics (87% believe their data is ready; 43% name readiness as the biggest barrier) are doing rhetorical work that the contradiction itself makes suspect: both numbers can't reflect careful measurement. And the "Adaptive Gold" tier — agents curating their own materialized views — is flagged as early-days speculation, correctly, but sits oddly among otherwise battle-tested patterns.

What's left out: cost. A stack of contracts, quarantine gates, lineage spans, three versioned models, and curated capability declarations is an expensive operating model, and the article's "who owns all this?" section names the ownership problem without pricing it. For a mid-size company, the honest question is whether a thinner version — contracts on the five datasets that matter, one semantic model, read-only access — captures most of the value. The article's own "start narrow" advice implies yes, but the maturity table's agent-ready column describes something close to a full platform team's output. The gap between the starting advice and the end-state table is where most readers will actually live.

## Related

- [[Beyond the Warehouse — Data Stacks That Actually Work]] — Strengthens this article's context-layer thesis from the practitioner side: Thomas in 't Veld's "narrow waist" domain model and semantic-layer-as-agent-interface is essentially this article's domain and semantic models arrived at independently, and his security-boundary framing anticipates the capability model's governed access.
- [[Inside OpenAI's In-House Data Agent]] — Strengthens the central claim with a deployed counterfactual: OpenAI's own write-up concluded the hard problem was making 600PB of data legible to an agent, not model intelligence — exactly the "context over models" finding this article generalizes, including the same semantic-layer mechanism.
- [[The Agent Access Model]] — Complements the governance section at greater depth: Cloudflare's five principles and Trust Ratchet address the same delegated-access and least-privilege territory, and the ratchet's in-flight capability narrowing is a more dynamic answer to the staged-autonomy ladder this article presents as a static table.
- [[Vedana — Domain Models as Agent Context]] — Nuances the three-model context layer: Vedana treats the domain model itself as the agent's primary context, which this article would say is only one-third of the layer — useful for testing whether the domain/semantic/capability split holds in practice or collapses into one artifact.

---
*Sources: [[raw/making-data-ready-for-agentic-ai-html]], [[summary/making-data-ready-for-agentic-ai-html]]*
