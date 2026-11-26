---
url: https://ai.engineer/code/2026/talks/online-aws-specification-analyzer
title: "Online AWS Specification Analyzer"
author: ai.engineer (talk summary of Rajath Kotyal's talk)
date_fetched: 2026-11-26
date_published: 2026
topics:
  - specifications-as-the-product
  - guardrails-and-feedback-loops
---

A talk summary of Rajath Kotyal's proposal to run the design review *before* code generation: inspect the specification itself for contradictions, gaps, and ambiguity, using an LLM to translate prose requirements into a formal representation and an SMT solver to check whether the resulting constraints can hold together. The motivating example is an e-commerce spec where one sentence mandates backorders when inventory is unavailable and another forbids backorders entirely — each sentence fine on its own, together an impossible instruction for a coding agent.

The pipeline has three moves: refine the wording, translate requirements into formal statements, and analyze that representation for conflicts, gaps, and scenarios. Kotyal is careful about the limit of this: formal checking proves consistency of the *translation*, not fidelity to the author's intent — an LLM can convert an ambiguous sentence into one perfectly consistent reading while silently discarding another. So a second check samples multiple translations of the same requirement and clusters them by logical equivalence; disagreement among clusters becomes an explicit decision point rather than a silent guess. The "remove the record" example shows the stakes: delete the data, or retain it and hide it from users — both are plausible implementations with very different consequences for retention and recovery.

The conclusion keeps humans where they belong: there is no automatic oracle for what a specification should mean. The solver can expose a contradiction or show that interpretations disagree; it cannot decide whether the business wants backorders. The division of work is the takeaway — the LLM translates, the solver checks, and a human makes the consequential choice before the implementation exists.
