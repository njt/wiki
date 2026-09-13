# NIST AI RMF Core (2023)

Section 5 of the NIST AI Risk Management Framework 1.0 — the "Core" of govern, map, measure, and manage: four continuous, cross-linked functions with ~60 outcome subcategories that turn AI risk management from an aspiration into an organizational operating system. This is the primary text behind most secondary AI-governance writing, and it predates autonomous coding agents; a revision is in progress, so every claim here should be dated to the January 2023 text.

---

## What it argues

AI risk management is a loop, not a phase. **Govern** builds the culture, policies, and accountability that "enables the other functions"; **Map** establishes context and feeds a go/no-go decision; **Measure** runs TEVV against trustworthiness characteristics; **Manage** prioritizes, treats, and monitors the residual risk, including decommissioning and incident response. Each function explicitly feeds the next, and all of it is meant to run "continuously, timely, and performed throughout the AI system lifecycle."

Two design choices give the document its character. First, it is deliberately *not* a checklist — "Actions do not constitute a checklist, nor are they necessarily an ordered set of steps" — even though every subcategory is phrased as a checkable end-state ("are in place and documented"). Second, it is organizational to the bone: senior leadership sets risk tone, roles are documented, third-party supply-chain risk appears in all four functions, and affected communities get feedback and appeal channels.

## Key quotes

> Actions do not constitute a checklist, nor are they necessarily an ordered set of steps.

The framework's own hedge, immediately followed by ~60 checkbox-shaped subcategories. Auditors will treat them as a checklist anyway — this is the same gap between intent and artifact that [[Chiaro Methodology]] resolves by making controls explicit, stable-referenced machine-readable data rather than prose outcomes.

> Risk management should be continuous, timely, and performed throughout the AI system lifecycle dimensions.

The load-bearing sentence. Everything else in the framework is a consequence of treating risk as a loop rather than a gate — the same conviction that runs through [[Guardrails and Feedback Loops]], transposed from code review to organizational governance.

> The interdependencies between these activities, and among the relevant AI actors, can make it difficult to reliably anticipate impacts of AI systems... the best intentions within one dimension of the AI lifecycle can be undermined via interactions with decisions and conditions in other, later activities.

The Map function's honest premise: no one has full visibility, so context-gathering is a defense against your own pipeline. This is a governance-grade statement of the local-optima problem that practitioner essays keep rediscovering.

> The risks or trustworthiness characteristics that will not – or cannot – be measured are properly documented. (Measure 1.1)

The most quietly radical subcategory in the document. It concedes that measurement is incomplete and demands the gaps be named — an admission you will not find in vendor evals. It is the honest counterpart to [[Goodhart's Law and AI Benchmarks]]: metrics get gamed precisely where nobody documents what was left unmeasured.

> The AI system to be deployed is demonstrated to be safe, its residual negative risk does not exceed the risk tolerance, and it can fail safely, particularly if made to operate beyond its knowledge limits. (Measure 2.6)

"Beyond its knowledge limits" is doing real work here — 2023 language for distribution shift that reads, in 2026, as a description of an agent leaving its competence envelope. The framework saw the failure mode; what it did not see is how fast the actor would gain hands.

## Key themes

- **#concept** Four-function loop — govern → map → measure → manage, with each function feeding the next and govern as the cross-cutting substrate
- **#concept** Trustworthiness as a measurable characteristic set — safety, security, resilience, privacy, fairness, explainability, transparency, environmental impact, each with its own Measure subcategory
- **#pattern** Documenting the unmeasurable — Measure 1.1's requirement to name what you cannot measure, the framework's most transferable idea
- **#pattern** Third-party and supply-chain risk recurring in every function — Govern 6, Map 4, Manage 3 treat pre-trained models and third-party data as first-class risk objects
- **#comparison** Voluntary framework vs. binding regulation — NIST positions this as adaptable voluntary guidance, with the Playbook as its tactical companion

## Critical analysis

Read in 2026, the framework is a period piece with excellent bones. Its unit of analysis is the *AI system* as a product — Map 2.1 names "classifiers, generative models, recommenders" — written for organizations with governance boards and vendors. It does not model an agent that writes code, calls tools, and escalates its own permissions inside a developer's environment; there is nothing here about prompt injection, harness design, or the observation-vs-action boundary that consumes the current security literature. That is what the in-progress revision has to catch up with, and readers should treat every organizational claim here as 2023-dated.

The deeper tension is between the "not a checklist" disclaimer and the subcategory format. Every statement is a declarative end-state — "processes are in place," "mechanisms are established" — which is exactly the shape a paper-compliance industry feeds on. An organization can be fully "compliant" with all sixty outcomes while shipping risky systems, because the framework audits the presence of processes, not their efficacy. The interesting modern question is what it would mean to make these outcomes falsifiable — [[Chiaro Methodology]] shows the pattern for SOC 2: stable control IDs, test attributes with pass criteria, calibration examples encoding judgment. Nobody has done that for the AI RMF yet, and until someone does, the framework's honesty about unmeasurables (Measure 1.1) is more valuable than its complete-looking tables.

What survives best is the shape, not the specifics. The four-function loop maps cleanly onto individual agent practice at smaller scale: map is spec-and-context work, measure is evals and verification, manage is rollback and incident response, govern is the steering files and permission policy you write before the first run. And its insistence on independent review ("internal experts who did not serve as front-line developers") and on affected-party feedback channels are the two pieces practitioner literature most often drops. As the primary source behind the framework taxonomy in [[AI Security Framework for DevSecOps]], this is the document everyone else is summarizing — worth reading in the original once.

## Connections

- [[AI Security Framework for DevSecOps]] — cites the NIST AI RMF as the "voluntary governance" layer of its framework taxonomy; this note supplies the primary text behind that citation, and shows exactly which half (govern, map) the DevSecOps piece had to compress to get to pipeline controls.
- [[Chiaro Methodology]] — operationalizes audit controls as machine-readable data with stable IDs and calibration examples; it is the existence proof for what the RMF's prose subcategories would need to become before agents could execute or verify them.
- [[Eval-Driven Development (Airbnb)]] — the practitioner-scale realization of the Measure function: Airbnb's three-layer evaluation toolkit and trajectory evaluation are what "rigorous TEVV with documented metrics" looks like when a product team, not a nation-state, does it.
- [[Agentic AI Security Stack]] — maps threats to OWASP/MITRE ATLAS/MAESTRO at the attack layer; the RMF is the governance layer above it, and Manage 3's third-party monitoring is the control the security stack's kill chains assume exists upstream.

---
*Sources: [[raw/5-sec-core]], [[summary/5-sec-core]]*
*Last updated: 2026-09-13*
