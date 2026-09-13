# When Spec-Driven Development Pays Off

An InfoQ report on the author's GAISS 2026-accepted study of specification governance for AI-generated code, and the rare spec-driven piece that measures its own sales pitch. The measured result is a trade-off, not a win: an approved spec+HLD+LLD baseline did not make drift reviewers better bug-finders (recall 0.525 vs 0.518, p=0.69), but it took findings from 0% attributable to 81% attributable (p=0.043) at a 48-vs-27-minute time cost. Delivery beats presence (staged SPEC.md-then-implement doubled the pass rate over inline spec-then-code prompting), easy-task spec gains are mostly a reasoning effect in disguise, and the payoff is a targeting rule: spend governance only on hard, multi-constraint, regulated work built by capable-but-imperfect models.

---

## Key Quotes

> "A specification baseline did not help reviewers catch more bugs. Rather, the baseline made the bugs they caught accountable."

The honest headline, stated plainly against the intuitive selling point of the whole movement. The value of a spec is not detection — reviewers found the same defects either way — it is ownability: findings a named reviewer can tie to an approved requirement, defend, and hand off.

> "Code-only reviewers kept hedging that they 'could not tell whether the behavior was intended'. That sentence is the whole governance gap in a single line."

With no contract, "something looks off" has nowhere to land. This is also exactly what the 90-review LLM replication showed: machine reviewers' attribution collapsed to 0.00 without a baseline, consistently across all three models. The gap is structural, not human — there is no requirement to cite when nothing was approved.

> "On easy tasks, the dramatic gains people attribute to 'specifying first' are largely a reasoning effect in disguise. If you are going to claim specification prompting improves quality, control for reasoning first."

The piece's sharpest methodological contribution, and the deflationary one. A chain-of-thought control (reason about edge cases, write no spec) captured almost all of the spec-first gain on single-function tasks (59% → 95% reason-first vs 92% spec-first). Most spec-first testimonials in the wild are uncontrolled for this.

> "On a twenty-invariant banking service... 'a specification, then the code' in a single prompt... was indistinguishable from just asking for code directly... A staged arm, author the specification, then implement from it in a fresh generation step, nearly doubled the pass rate to forty-five percent."

Delivery beats presence: same words, two deliveries. Only the one treating the spec as a governing artifact rather than inline prompt text moved the outcome. The artifact-hood is the mechanism, not the writing.

> "The model is Responsible (R) for generation but never Accountable (A)... Accountability always rests with a human."

The RACI naming is what makes the audit trail meaningful rather than decorative: each AI-generated behavior traces to an approved requirement and an accountable human decision — the part conceptual governance frameworks (EU AI Act, ISO/IEC 42001, NIST AI RMF) usually leave undefined.

> "Make the code abundant. Just don't make the plan optional. Don't pretend the plan is free."

The closing is a triple: governance yes, ceremony no, and name the tax. The 48-minute review cost is the tax — predictable, per-output, and the most automatable part of the loop via an LLM pre-screen.

## Key Themes

#concept #spec-driven #governance #pattern

- **Verification is the new bottleneck** — AI raised code volume without raising confidence; the hard question becomes a governance question (who is accountable, how do you detect divergence, how do humans and models split oversight labor).
- **The spec as governance artifact, not prompt decoration** — spec + HLD + LLD form one versioned baseline; five lifecycle control points each leave an artifact behind (approved baseline, generation record, drift log, reconciliation record), and the audit trail falls out of the loop as a by-product.
- **Recall null, attribution effect** — the baseline buys accountability, not detection. A governance regime wants findings a named reviewer can tie to an approved requirement, not more findings.
- **Delivery beats presence** — a staged SPEC.md-then-implement step nearly doubled pass rates where inline spec-then-code prompting did nothing.
- **The reasoning confound** — easy-task spec gains are mostly deliberation, not specification; any spec-first claim needs a reason-first control arm.
- **The targeting rule** — hard, multi-constraint, regulated, long-lived systems built by capable-but-imperfect models (weak model +21 points on the banking task, strong model +2); skip it for one-shot tasks and throwaways.

## Analysis

This is the most empirically honest document in the wiki's spec-driven cluster, and its value comes precisely from what it refuses to claim. Every other spec-first source in this wiki — the workflows, the case studies, the manifestos — argues forward from enthusiasm. This one built a study around the single act at the center of the whole idea (a human reviewing AI code against an approved baseline), reported that its intuitive selling point failed to replicate on its own terms, and kept the finding anyway. "Governance work that hides its own uncertainty is not governance" is a sentence most of the cluster could not have written.

The recall null matters more than it looks. The intuitive pitch — "review the spec first, catch more bugs" — is dead on this evidence. What survives is narrower but more durable: attribution. A finding tied to `transfer_idempotent` can be owned, defended, and audited; a finding described as "something looks off" cannot enter an accountability trail at all. The 90-review LLM replication makes the point structural: machine reviewers show the identical pattern, which means LLM code-review tools are a fine pre-screen but cannot produce accountability evidence without a governed baseline — they can flag issues and still tie none of them to a requirement. That quietly reframes the AI-review-tool market: the bottleneck product is not the reviewer, it is the approved contract.

The reasoning confound is the piece's sharpest gift to the rest of the cluster. If chain-of-thought captures almost all of spec-first's gain on easy tasks, then a large fraction of practitioner testimonials for spec-first tooling are measuring "the model thought longer," not "there was a document." This should make everyone in the spec-driven space — including the enthusiasts for heavyweight SDD pipelines — add a reason-first control arm before attributing wins to their favorite artifact. It also sharpens the targeting rule into something testable: the spec discipline earned its keep exactly where raw deliberation was insufficient, i.e. where many constraints must stay mutually consistent and the model had room to fail (~21 points for the weak model, ~2 for the strong one).

The honest limits deserve to be carried forward rather than dropped: five reviewers, two services, one domain; the paired test bottoms out at p=0.043 at n=5, so the "significant" results certify direction, not effect size; and the staged-generation win carries a second-pass confound (the winning arm runs the model twice). Read it as a strong pilot signal. The obvious objection — "so specs don't help find bugs, why pay 48 minutes?" — is answered by the piece itself but worth stating: the question assumes review is for detection, when for regulated systems review is for evidence. What the 21 extra minutes buy is a trail a regulator, an auditor, or the next engineer can follow.

**How this source relates to the wiki:**

- [[Spec-Driven Development]] — strengthens Breunig's triangle (spec/test/code synchronization) with the first controlled measurement in the cluster, and complicates it: the synchronized baseline did not raise recall, and easy-task gains are mostly reasoning — so the triangle's payoff is governance and attributable findings, not bug-finding.
- [[Principal Drift]] — supplies the measured counterpart to O'Reilly's task-routed governance: a named, versioned baseline is the concrete mechanism that makes drift findings attributable, and the targeting rule (govern only hard, multi-constraint, regulated work) is a risk-tiering rule for where to spend review effort.
- [[Specsmaxxing]] — the twenty-invariant drift contract is the studied, peer-reviewed version of ACID-threaded acceptance tracking; this piece quantifies the cost Specsmaxxing doesn't name (48 vs 27 minutes per review) and adds the structural caveat that attribution requires the baseline to actually be approved and versioned.
- [[SDD Case Study — 13 Apps in 70 Days]] — Fontoura's "correct the spec, not the code" discipline gets a measured grounding here: the staged spec-then-implement delivery is what moved outcomes (delivery beats presence), and the audit-trail logic explains *why* the spec corpus is the artifact worth maintaining across sessions.

---
*Sources: [[raw/when-spec-driven-development-pays-off]], [[summary/when-spec-driven-development-pays-off]]*
*Last updated: 2026-09-13*
