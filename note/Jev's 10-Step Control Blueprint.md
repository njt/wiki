# Jev's 10-Step Control Blueprint

0xwhrrari's promotional thread distilling Diogo Almeida's 12-page Jev PDF into a 10-step blueprint for wrapping a deterministic decision layer around LLM calls: split responsibilities, build compact state, use typed primitives, sandwich the LLM between Jev before and after, route by confidence, batch decisions, and log decision receipts.

---

## The 10 steps, compressed

1. **Split responsibilities** — the LLM generates; Jev makes bounded semantic decisions; deterministic code keeps authority.
2. **Build the state** — give the decision layer current request, evidence, policy, and proposed action, not the whole conversation.
3. **Choose the right primitive** — Choice selects a route, Score evaluates an ordered rubric, Noul returns the probability a statement is true.
4. **Atomic questions** — intent, urgency, evidence, risk, and scope become separate typed decisions instead of one giant evaluation prompt.
5. **Jev before the LLM** — select context, tools, provider, and workflow before paying for an expensive generative call.
6. **Bounded LLM jobs** — once the route is selected, the model gets only what that branch requires.
7. **Jev after the LLM** — verify the result answers the request, uses sufficient evidence, and stays in scope.
8. **Route by confidence** — high-confidence/low-risk auto-proceeds; uncertainty requests more context; consequential actions go to review.
9. **Batch independent decisions** — many Choice/Score/Noul questions over one shared state, not one LLM call per judgment.
10. **Record the decision receipt** — state version, question, probabilities, route, model, latency, outcome, human override.

The tagline does the positioning: "most AI courses teach you how to write a bigger prompt; this teaches you how to build the control system around every prompt."

---

## Key Quotes

> "the LLM generates, Jev makes bounded semantic decisions and deterministic code keeps authority"

The whole philosophy in one clause — a three-way division of labour where generation is the *least* trusted of the three parties. It inverts the usual architecture: instead of an agent that occasionally calls a checker, you get a control system that occasionally calls a generator.

> "give Jev the current request, relevant evidence, policy and proposed action instead of sending the entire conversation"

Context discipline as a first-class step. This is the same insight as [[Layer-First Pattern — Keep Data Out of the LLM Context]] applied to decisions rather than data: state should be *authored*, not accumulated.

> "high-confidence low-risk cases proceed automatically, uncertain cases request more context and consequential actions go to review"

Step 8 is the money step, and the one that generalises beyond Jev entirely — it's a confidence-gated autonomy policy, the same shape as Meta's risk-calibrated review funnel in [[RADAR — Meta's Risk-Aware Diff Auto Review]].

> "record the complete decision receipt: state version, question, probabilities, selected route, model, latency, outcome and human override"

A decision receipt is the audit-grade version of "what a useful AI trace should contain" — every judgment becomes queryable after the fact. That's what makes the control system *improvable* rather than merely *imposed*.

---

## Key Themes

- #pattern — **Sandwich the LLM.** Jev both before (routing, context selection) and after (verification) the generative call. The LLM becomes a bounded worker inside a deterministic control loop, not the loop itself.
- #concept — **Typed decision primitives.** Choice/Score/Noul map judgments to well-typed, cheap, inspectable operations — routing, rubric scoring, and probability, each a separate call with its own semantics.
- #tool — Jev (TypeSafe), the "System One" fast decision layer; the PDF is the practitioner's manual for it.
- #pattern — **Decision receipts.** Full provenance per judgment: inputs, probabilities, route, model, latency, outcome, override. Audit and improvement both run over the receipt log.

---

## Analysis

As marketing, this is a competent condensation — the thread gives you the architecture in ten numbered moves without once saying "buy." As engineering, it's a checklist of already-argued positions (bounded contexts, atomic questions, confidence gating, provenance logs) finally assembled around a *product* designed for exactly those moves, which is arguably its real contribution: the blueprint names no trick that couldn't be implemented with a small classifier and plain code, so Jev's value proposition has to be the primitives themselves and their latency. The sceptical reading — cf. [[Jev Can't Be Calibrated]] — is that "Noul returns the probability that a statement is true" smuggles in the calibration claim the wiki's other sources dismantled; the blueprint's steps 5–9 quietly *depend* on those probabilities meaning something, which is the load-bearing assumption this thread never states. Still, as a description of where agent architecture is heading — expensive generation as a rare, bounded, verified event inside a cheap deterministic control system — it's a sharp ten-line summary, and step 10 alone is worth copying into any agent harness.

---

## Relations

- Strengthens [[Jev's Architecture Unmasked]]: the forensic teardown explains *how* the model works behind the primitives this blueprint takes as given — shared-state batching and typed readouts map directly to steps 3 and 9.
- Complicates [[Jev Can't Be Calibrated]]: step 8's confidence routing presupposes usable probabilities, precisely what Molas argues a vendor cannot guarantee across distributions.
- Reinforces [[Deterministic Spine, Agentic Leaves]] with a concrete, productised version of the deterministic-spine pattern — the spine here is a purchased decision model rather than linters and schema checks.
- Nuances [[What a Useful AI Trace Should Actually Contain]]: the "decision receipt" is a crisp, operational answer to what an audit trace of agent *judgments* (not just actions) should hold.

---
*Sources: [[raw/2103157617715491278]], [[summary/2103157617715491278]]*
*Last updated: 2026-10-11*
