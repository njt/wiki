# Online AWS Specification Analyzer

Rajath Kotyal's talk (summarised on ai.engineer) proposes moving the first review earlier: inspect specifications for contradictions, gaps, and ambiguity *before* handing them to the coding agent, using an LLM to translate requirements into formal statements and an SMT solver to check whether those statements can hold together — with sampled, clustered translations used to surface ambiguity as an explicit human decision rather than a silent guess.

---

The motivating example is deliciously mundane: one sentence of an e-commerce spec says an order becomes a backorder when inventory is unavailable; another says no order should ever become a backorder. Each is fine alone; together they are an impossible instruction. Better code generation cannot fix that, because the conflict lives in the requirements, not the code.

> "Better code generation cannot settle which business rule the author actually meant."

This is the sharpest sentence in the summary, and the whole argument in one line. The failure mode is not model quality but upstream incoherence — which is why Kotyal's move is to put the check *before* generation rather than after.

> "An LLM can turn an ambiguous sentence into one perfectly consistent interpretation while silently discarding another plausible reading."

The honest limit of the approach, stated plainly: formal checking verifies the translation, not the intent. The countermeasure is not more translation but comparison — ask for the same requirement's translation multiple times and cluster by logical equivalence. Superficially different outputs can be semantically identical; superficially similar ones can hide different assumptions. Disagreement becomes a decision point, not a bug found later in a codebase.

> The tool should present the choice rather than settle it accidentally through code.

The "remove the record" example: delete the data, or retain it and hide it? Both produce plausible implementations; they differ on retention, recovery, and future queries. This is exactly the class of decision that [[silently-resolved-ambiguity-is-comprehension-debt-of-intent]] warns gets buried in code as invisible comprehension debt.

## Key themes

#concept #tool #pattern

- **LLM translates, solver checks** — a clean division of labour: probabilistic translation into formal representation, deterministic SMT checking of the relationships between the resulting statements.
- **Consistency ≠ intent** — the pipeline can prove the represented rules cohere, but cannot decide whether the business wants backorders. No automatic oracle for meaning exists.
- **Abstention as an outcome** — the system can accept, ask the author, or abstain from implementing. Refusing to guess is a feature.
- **Automating the design review** — the flow-and-requirements-reading ritual engineers already do, partially mechanised while questions are still about behavior rather than bugs.

## Analysis

This talk slots neatly into the wiki's formal-specification thread and strengthens its most important claim: that the value of formalisation sits upstream of code generation, not in verifying the output. [[Rewrite All the Code All the Time]] argues natural-language specs can never support fully automatic regeneration; Kotyal's pipeline is a concrete partial answer — you cannot make the spec fully formal, but you can *check the parts you managed to formalise* and route the residue to a human.

The multiple-translation-and-cluster trick is the most original move, and it nuances [[Your .md File Is Not a Specification]], which showed a model checker surfacing underspecified behavior from prose. FizzBee's example found a product decision hiding in two requirements; Kotyal's adds a method for *detecting that a decision exists* when a single translation looks clean — by exploiting the fact that sampling exposes which parts of the interpretation were arbitrary. It is an epistemic hedge: disagreement among samples is evidence of ambiguity even when every sample is self-consistent.

It also complicates [[SLCR]] and [[Spec Kit]], which treat the spec as a durable artifact to be checked against code. Kotyal's artifact is more provisional: a translation to be stress-tested and discarded, with the *author's decision* — not the formal text — as the thing worth recording. The cautious presentation of evidence (the summary notes the approach is "emerging") is a refreshing contrast to the usual pipeline-demo overclaiming; the design review automation framing gives it a credible, incremental adoption path.

---
*Sources: [[raw/online-aws-specification-analyzer]], [[summary/online-aws-specification-analyzer]]*
*Last updated: 2026-11-26*
