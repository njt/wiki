---
url: https://ai.engineer/code/2026/talks/online-aws-specification-analyzer
date_fetched: 2026-11-26
---

## Introduction

An e-commerce specification says that an order should become a backorder when inventory is unavailable. Elsewhere, it says that no order should ever become a backorder.

Each sentence is easy to understand on its own. Together, they ask the coding agent to implement an impossible combination. Better code generation cannot settle which business rule the author actually meant.

Rajath Kotyal's talk moves the first check earlier: inspect the specification before handing it to the agent that writes the implementation. The project he describes combines an LLM's ability to translate natural language with a solver's ability to check relationships between formal statements.

## Find the problem before it becomes code

The backorder example illustrates a conflict across sentences. Other problems can hide inside a single sentence: a vague verb, an unstated condition, or a level of detail that leaves the implementation to guess.

There are also gaps. A set of rules may cover submitted and completed orders while saying nothing about an idle order. Nothing has to be directly contradictory for the agent to be left making a decision the specification's author never made.

This is familiar work in design reviews. Engineers draw flows and read requirements together to discover cases that prose obscures. Kotyal's proposal is to automate part of that first review, surfacing questions while they are still questions about intended behavior rather than bugs embedded in a codebase.

## Let the LLM translate and the solver check

The pipeline begins by refining the wording. It then translates requirements into a formal representation and analyzes that representation for conflicts, gaps, and scenarios.

*At 01:54:20, the slide separates the LLM's translation work from the solver's checking work. It runs on requirements before code generation.*

An SMT solver checks logical constraints expressed in a formal language. For the order example, the important relationship is between the condition “submitted with no available inventory” and the required backorder state, together with the global prohibition against that state. Expressing those relationships makes it possible to ask whether the relevant constraints can hold together.

That precision applies to the formalized requirements. It does not prove that the translation captured the author's intent. An LLM can turn an ambiguous sentence into one perfectly consistent interpretation while silently discarding another plausible reading.

Kotyal therefore adds a second kind of check: compare translations rather than trusting the first one.

## Sample interpretations, then compare their meaning

At 01:55:00, Kotyal describes asking the model to translate the same requirement multiple times, then clustering the outputs by logical equivalence.

Different wording is not necessarily disagreement. Two formal expressions can mean the same thing. Conversely, superficially similar outputs can encode different assumptions. Clustering by meaning is meant to separate those cases.

*The 1:55:40 slide turns model disagreement into an explicit decision point instead of letting one arbitrary interpretation reach the codebase.*

The next action depends on the result. The system can accept a sufficiently clear interpretation, ask the author to resolve the ambiguity, or abstain from implementing the requirement. Repeating a translation is useful because it exposes alternative assumptions, not because repetition converts generated text into ground truth.

“Remove the record” is the talk's concrete example. Does that mean deleting the data, or retaining it while hiding it from users? Both can produce plausible implementations. They have different consequences for retention, recovery, and future queries. The tool should present the choice rather than settle it accidentally through code.

## The author still owns the intended behavior

The ending puts a limit on the entire pipeline. There is no automatic oracle for what a specification should mean. Domain knowledge can be implicit, and even a logically consistent specification can describe the wrong product.

Formal checking is useful precisely when its scope stays clear. It can expose a contradiction in the represented rules or show that the sampled interpretations disagree. It cannot decide whether the business wants backorders, whether “remove” should preserve a legal record, or whether an omitted state is intentionally outside scope.

Kotyal also describes the evidence for this emerging approach cautiously. The useful takeaway is a division of work: the LLM translates, the solver checks, and a human makes the consequential choice when the intended rule is unresolved.

That gives the coding agent a better starting point. Instead of asking it to repair code written from conflicting instructions, resolve the contradiction before the implementation exists.
