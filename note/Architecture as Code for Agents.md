# Architecture as Code for Agents

A short Dotneteers essay arguing that code is no longer the whole truth of a system for agents: the *why* behind boundaries, data isolation, resilience levels, and regulatory workarounds lives outside the repo, and agents that read every line will still miss it. The fix is architecture-as-code — machine-readable intent (FINOS CALM) validated before changes are accepted.

---

The essay's premise is a knowledge-location problem. A repository answers "what does this system do?" but not "why is this boundary here?" — and the answer to the second question is exactly what an agent needs to avoid "fixing" an intentional constraint. That context sits in diagrams, documents, and heads; humans already struggle to find it, and agents may never know to look.

The proposed remedy is to promote architecture from documentation to specification: FINOS CALM's components, relationships, controls, and constraints give agents machine-readable intent that can be validated before a change is accepted. The author's sharpest formulation is the new question agents should be asked:

> "𝘋𝘰𝘦𝘴 𝘵𝘩𝘪𝘴 𝘤𝘩𝘢𝘯𝘨𝘦 𝘴𝘵𝘪𝘭𝘭 𝘤𝘰𝘯𝘧𝘰𝘳𝘮 𝘵𝘰 𝘵𝘩𝘦 𝘢𝘳𝘤𝘩𝘪𝘵𝘦𝘤𝘵𝘶𝘳𝘦 𝘸𝘦 𝘪𝘯𝘵𝘦𝘯𝘥𝘦𝘥?"

Not "does the code compile?" but a conformance check against intent — which reframes review as constraint validation rather than reading.

The closing move is deliberately organizational:

> "It will need organizations capable of expressing architecture in a form both humans and machines can challenge."

The bottleneck isn't the validator; it's that most organizations cannot state their architecture precisely enough for anything — human or machine — to challenge it.

## Themes

- #concept — intent beyond the code: the "why" as a first-class, agent-readable artifact
- #tool — FINOS CALM as concrete architecture-as-code machinery
- #pattern — validation-before-acceptance: conformance checks as a review gate

## Analysis

This is a compressed statement of an idea the wiki keeps circling: when agents write the changes, the durable artifact must be the spec, not the diff. Its distinctive contribution is naming *why*-knowledge as the gap — a compliance-motivated inefficiency looks like a defect to an agent with no access to the constraint, and the agent will "fix" it confidently. That makes architecture-as-code a safety mechanism, not just documentation hygiene.

The essay is thin on mechanism — it names CALM and gestures at validation but shows no pipeline. Read alongside [[Architectural Guardrails for AI-Generated Code]], which works out the enforcement side in detail, it reads as the thesis statement for a practice the wiki has already operationalized. Its honest admission is that this is an organizational capability problem before it is a tooling problem, which matches [[How to Prepare for AI-Driven Code Modernization Projects]]' finding that mobilizing the organization around changes, not producing them, is the bottleneck.

It also sharpens the ADR debate: [[Coding Agents Love Decision Records]] treats decision records as agent context that can go stale and needs curation; architecture-as-code proposes the same move for structural intent, but in a form a machine can *check* rather than merely read. And it connects to [[Agents and Acquiring Debt]]'s comprehension debt: without a machine-readable record of intent, every agent change silently erodes the why, and nobody signs for the loss.

## Related pages

- Strengthens [[Architectural Guardrails for AI-Generated Code]] with the motivation layer: the constraint an agent can't see is the one it will break.
- Nuances [[Coding Agents Love Decision Records]] — from prose records agents read to structured models agents can be validated against.
- Complicates [[Agents and Acquiring Debt]] — unrecorded architectural intent is comprehension debt with no ledger at all.
- Sits within [[Specifications as the Product]] — architecture-as-code is the spec-driven thesis applied to structure rather than features.

---
*Sources: [[raw/ai-agents-can-read-every-line-of-your-code]], [[summary/ai-agents-can-read-every-line-of-your-code]]*
*Last updated: 2026-10-08*
