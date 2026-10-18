# Agentic Property-Based Testing

An arXiv workshop paper (Anthropic) describing a Claude Code agent that mines property-based tests from Python codebases at ecosystem scale: 933 modules across 100 packages, 984 bug reports at 56% validity, ~$9.93 per real bug, and three upstream patches merged — the largest systematic demonstration that LLM-guided PBT is a rigorous, scalable bug-finding method.

---

The design is almost aggressively plain. The "agent" is a Markdown prompt on Claude Code with a six-step loop — analyze target, understand it, propose properties, write Hypothesis tests, execute and triage, report — plus a to-do list and broad autonomy. The authors are explicit that the contribution is not a novel architecture but the evaluation: proof that LLM-plus-PBT works at scale with few false alarms. The engineering weight sits in the anti-false-alarm discipline: properties must be grounded in documentation and call sites, failing tests go through a reflection rubric, and only convinced findings become reports.

The economics are the headline: $5,474 of API spend for 136.6 hours of agent time, yielding ~$9.93 per valid bug — an upper bound, since a developer targeting the agent judiciously would do better. The rubric pipeline matters as much as the raw hit rate: naive reports were 56% valid, but model-scored ranking lifted the top tier to 86% valid and 81% "worth reporting." Triage, not generation, is where the quality came from.

---

## Key quotes

> "Our agent is built on top of Anthropic's Claude Code... Our agent is implemented as a natural language prompt, stored in a Markdown file... As it is a prompt, our agent is easily portable to other agent implementations."

The whole system is a prompt. No fine-tuning, no custom harness — portability is treated as a feature, and the evaluation is the artifact.

> "The bugs span diverse failure modes from serialization failures to numerical precision errors to flawed cache implementations."

The range is the argument for PBT itself: the properties (round-trip, non-negativity, hash distinctness, "Easter is a Sunday") are domain-agnostic shapes that catch domain-specific bugs.

> "the agent cannot distinguish intentional design violations from bugs"

The honest limit. The requests `LookupDict` report was a duplicate of a complaint a maintainer rejected in 2022 — the property was inferable from the name but wrong about intent. Specification remains a social act.

---

## Themes

#concept #tool #pattern

---

## Analysis

Compared to the Anthropic blog write-up of the same work, this arXiv version adds the mechanical details practitioners actually want: the exact prompt structure, the sampling methodology (933 modules, three selection strategies), the cost accounting, and the related-work positioning (Vikram et al.'s 41% pass rate for one-shot PBT generation vs. this agentic loop). The multi-step loop is what closes that gap — not a better model, but iteration with reflection.

The paper also quietly validates a thesis running through this wiki: verification beats generation. The agent's value comes from an unfakeable oracle (a failing property test is a counterexample, not an opinion) plus a triage stack for handling what the oracle surfaces. The dual-use caveat — autonomous vulnerability discovery — is noted but understated; a $10-per-bug autonomous auditor is a significant shift in attacker economics regardless.

---

## Related pages

- [[Finding Bugs with Property-Based Testing]] — the Anthropic blog version of the same research; this paper is the fuller arXiv treatment with the prompt, methodology, and cost detail the blog post compresses.
- [[Agentic Verification]] — Osmani's property-tests-as-constraints argument in practice: this paper is the strongest empirical evidence that LLM-inferred properties make a workable oracle.
- [[A New Era for Software Testing]] — situates agentic PBT within the broader shift of testing from human-authored examples to machine-searched invariants.
- [[The Coming Need for Formal Specification]] — the intent-ambiguity limitation (dateutil's Julian Easter, requests' LookupDict) is exactly the boundary where inferred properties need explicit, human-owned specification.

---
*Sources: [[raw/2510-09907v1]], [[summary/2510-09907v1]]*
*Last updated: 2026-10-18*
