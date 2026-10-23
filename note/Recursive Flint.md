# Recursive Flint

Recursive's Flint is a 39.8B-total / ~2.5B-active mixture-of-experts model trained from scratch specifically for coding agents, released Apache 2.0 with an SGLang serving kit: 54.9% Terminal-Bench 2.1, 89.7% SWE-bench Pro v2, 74.2% SWE-bench Verified, 262K context, one B300/H200 GPU per replica.

---

The pitch is not "another general model that happens to code" but a model shaped around the agent loop itself: multi-turn tool use, reasoning traces preserved as `reasoning_content` across turns, and evaluation run through agentic harnesses (mini-swe-agent, Terminus 2) rather than chat benchmarks. The architecture choices read as long-context agentic work in miniature: a 513-token sliding window on 35 of 49 layers with only 14 full-attention layers, 256 routed experts with a shared expert always active, and an output head counted in the active-parameter figure.

The card is candid in two ways worth noting. First, it discloses an evaluation-honesty adjustment: Flint's Terminal-Bench score counts solutions obtained through web retrieval as failures, while developer-reported comparison scores "may not apply the same adjustment" — Recursive quietly accusing the field of benchmark inflation while inflating nothing itself. Second, the limitations section states plainly that "structured tool-call grammar does not prevent the model from choosing an unsafe or incorrect action," pushing permission enforcement into the application layer — a textbook restatement of the guardrails argument that grammar is not a boundary.

---

## Key Quotes

> "In our release evaluations, it scores 54.9% on Terminal-Bench 2.1, 89.7% on SWE-bench Pro v2, and 74.2% on SWE-bench Verified."

The headline numbers, and the composition matters: SWE-bench Pro v2 at 89.7% is the striking one, since Pro is the harder, contamination-resistant variant. But the harness is disclosed (mini-swe-agent) and the sampling settings are fixed — the card treats evaluation setup as part of the claim, which is rarer than it should be.

> "Flint's Terminal-Bench score counts solutions obtained through web retrieval as unsuccessful; developer-reported scores may not apply the same adjustment."

An entire benchmark-integrity debate compressed into one footnote. If agents can pull the answer off the web, Terminal-Bench stops measuring capability and starts measuring retrieval. Recursive unilaterally closing that hole — and noting that competitors may not — is both a methodology improvement and a competitive jab.

> "Tool-using applications should enforce their own permissions and execution boundaries; structured tool-call grammar does not prevent the model from choosing an unsafe or incorrect action."

The model-card equivalent of [[A Convention Is Not a Constraint]]: a valid tool call is not a safe tool call. This is the open-weights lab conceding that containment lives outside the model, in the harness.

> "Long reasoning can increase latency and exhaust the output budget before an answer is complete. The context configuration is a capacity limit, not a guarantee of uniform accuracy across all sequence lengths or tasks."

A 262K context sold as capacity, not competence — the same honesty about long-context degradation found in [[Context Rot]], now appearing in a vendor's own card.

---

## Themes

#concept #tool #pattern

- **Efficient-scale MoE for agents.** Flint sits in the same lane as [[Beam — Reflection's 501B Open-Weight Model]]: open-weight, Apache 2.0, positioned not against the frontier but on cost-per-resolved-task. At ~2.5B active parameters it is far more aggressive on sparsity than Beam's 23B — the workhorse-coding-agent niche is now a genuine product category.
- **Benchmarks as setup, not number.** Disclosing harness, temperature, attempt count, and the web-retrieval disqualification makes the scores auditable in a way most developer-reported comparisons are not.
- **Guardrails outside the model.** The responsible-use section hands containment to the application layer rather than implying the model's own tool grammar is a safety mechanism.

---

## Opinion

This is what a mature model card looks like — specifications, honest active-parameter accounting (2.5B including the output head, 2.2B for the body, footnoted), disclosed sampling settings, and limitations that admit real failure modes rather than boilerplate. The missing piece is the technical report: training data mix, RL details, and the mid-training curriculum are all summarized in one sentence, and the SWE-bench Pro number is measured by Recursive itself with the † convention marking which comparison scores are not. The claim to watch is the Terminal-Bench adjustment — if other labs adopt web-retrieval disqualification, some published agentic scores will fall; if they don't, cross-model comparison stays broken.

The strategic read: Recursive is selling the model for exactly the loop its own research ([[Harness2 (Recursive Agent Harnessing)]]) studies — and the combination of a cheap-per-token agent model plus self-improving harnesses is a coherent thesis about where coding-agent economics go. This release strengthens [[Beam — Reflection's 501B Open-Weight Model]]'s evidence that the open-weight coding-agent tier is crowding with efficient MoE specialists; it nuances [[Context Rot]] with a vendor conceding in its own card that long context is capacity rather than uniform accuracy; and it corroborates [[A Convention Is Not a Constraint]] from the model side — even a lab selling tool-calling models warns that tool-call grammar is not a permission system.

---

*Sources: [[raw/flint]], [[summary/flint]]*
*Last updated: 2026-10-24*
