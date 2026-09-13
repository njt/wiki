---
url: https://www.infoq.com/articles/when-spec-driven-development-pays-off/
title: "When Spec-Driven Development Pays Off"
author: "Not stated in fetched text (InfoQ article)"
date_fetched: 2026-09-13
date_published: "2026 (exact date not stated in fetched text)"
topics:
  - specifications-as-the-product
  - guardrails-and-feedback-loops
---

An InfoQ article reporting the author's controlled study (accepted at GAISS 2026) of specification governance for AI-generated code. The premise: AI made code abundant, so verification — not writing — is the new bottleneck, and the popular response of "review the spec, not just the code" deserves measurement rather than faith. The specification (plus high-level and low-level design) is treated as a governance artifact: a reviewed, approved, versioned baseline that every control point and accountability decision refers back to, structured as five lifecycle control points (authoring, review gate, guided generation, drift detection, reconciliation) whose artifacts — approved baseline, generation record, drift log, reconciliation record — form the audit trail that EU AI Act, ISO/IEC 42001, and NIST AI RMF compliance demands.

The empirical core is a drift-review study: five experienced reviewers, two AI-generated banking services (eleven and ten adjudicated ground-truth drifts), within-subject 2×2 design comparing review against an approved baseline versus review of code alone. The headline is a trade-off, not a win: recall did not improve (0.525 vs 0.518, p=0.69) — the baseline did not make reviewers better bug-finders — but attribution exploded from 0% to 81% (p=0.043): with a contract, every finding ties to a named invariant; without one, reviewers hedge that they "could not tell whether the behavior was intended". Baseline review also raised confidence (4.2 vs 3.4) and cost nearly double the time (48.4 vs 26.7 minutes). A 90-review replication with LLMs in the reviewer seat reproduced the exact structure: recall unchanged, attribution 0.67 vs 0.00 — without an approved contract there is nothing to cite, for human or model alike.

On the generation side the findings are deflationary. "Delivery beats presence": on the twenty-invariant banking task, a single prompt asking for "spec then code" was indistinguishable from asking for code directly (both 23.8% on a weak model), while a staged arm — author the specification, then implement from it in a fresh generation step — nearly doubled the pass rate to 45%. Same words, two deliveries; only the one treating the spec as a governing artifact rather than inline prompt text moved the outcome. And a chain-of-thought control (reason about edge cases, write no spec) captured almost all of the apparent spec-first gain on easy tasks (59% → 95% reason-first vs 92% spec-first): on easy work, "specifying first" is largely a reasoning effect in disguise.

The author is unusually honest about limits: five reviewers, two services, one domain, p floor of 0.043 at n=5, a second-pass confound on the staged-generation win. The conclusion is a targeting rule, not a universal prescription: specification governance pays in one corner — hard, multi-constraint, regulated, long-lived systems built by a capable-but-imperfect model (the weak model gained ~21 points on the banking task, the strong model ~2) — and is waste for throwaway scripts and tasks the assistant one-shots. A five-phase adoption path (pick the corner, stand up the gate, version the baseline, add drift detection, automate the walk) keeps the cost manageable.
