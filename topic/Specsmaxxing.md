# Specsmaxxing

Acai.sh's manifesto for structured spec-driven development using YAML-based acceptance criteria with stable IDs (ACIDs) that thread through code, tests, and a review dashboard. The core move: replace file-by-file PR review with requirement-by-requirement acceptance tracking. Born from one developer's self-described "AI psychosis" -- the loop of building harnesses to build harnesses -- and the realization that the only artifact worth maintaining is the spec itself.

---

## Key Quotes

> "Nothing beats an organic, pasture-raised, **hand-written** spec. Spec-writing is where the act of *software engineering* really happens."

This is the single most important sentence in the piece. It pushes back against the "AI-generated specs" trend that [[OpenSpec]] and others lean into. The author's position: if code is disposable, the one thing humans should still write by hand is the spec. That's where taste lives.

> "One hallmark symptom of AI psychosis is *using AI to build AI harnesses for building products*, rather than just using AI to build the damn product."

Honest self-diagnosis of the meta-tooling trap. Every practitioner in the [[Agent Coding Workflow]] space has felt this pull. The author threw out the branch and started over -- a rare admission.

> "The sky is not the limit. The context window is the limit."

The pragmatic version of what [[Agent Memory and Context]] catalogues academically. Context loss across sessions, machines, and handoffs is the failure mode that specs exist to mitigate.

> "In Acai.sh, specs describe how your system **SHOULD** behave. Current behavior is transient."

A direct shot at [[OpenSpec]]'s philosophy. This is a meaningful philosophical split in the spec-driven space: descriptive specs (what the system does) vs. prescriptive specs (what the system must do). Acai picks prescriptive.

> "If software were free and infinite and instant, your criteria for acceptability is really the only thing of value. The spec."

The thought experiment version of [[Specifications as the Product]]'s thesis. If code generation costs approach zero, the only scarce artifact is the expression of intent.

## Key Themes

#tool #spec-driven #concept #comparison

**ACIDs (Acceptance Criteria IDs).** The key invention. Each requirement gets a stable, hierarchical ID (e.g., `my-feature.AUTH.1-1`) that appears in the spec YAML, in code comments, and in test annotations. This creates a bidirectional traceability matrix between spec and implementation -- you can grep your codebase for any requirement and find exactly where it's satisfied.

**feature.yaml over markdown.** The author argues YAML is the right middle ground between unstructured markdown (too loose for machine parsing) and formal languages like EARS/Gherkin (too cumbersome for daily use). The format is a flat list of components with numbered requirements, plus constraints for cross-cutting concerns. Parsing support is the practical benefit; stable addressability is the real win.

**Acceptance coverage over test coverage.** A reframing: instead of asking "what percentage of code is tested?", ask "what percentage of requirements are implemented and verified?" This shifts the unit of measure from lines of code to units of intent.

**The specsmaxxing-to-testmaxxing-to-factory pipeline.** The article sketches a three-stage progression: (1) invest in specs, (2) invest in automated QA to validate against specs, (3) let agents react to red tests autonomously. This maps cleanly to levels 3-5 of [[Five Levels from Spicy Autocomplete to the Dark Software Factory]].

**The tool itself.** Acai.sh is a CLI + web dashboard (Elixir/Phoenix/Postgres, Apache 2.0). `acai push` sends specs and code references to the dashboard. Reviewers accept or reject individual requirements rather than reviewing files. Free hosted version, open source.

## Critical Analysis

The strongest move here is making [[OpenSpec]] look passive by comparison. OpenSpec's philosophy -- "specs describe how your system currently behaves" -- is descriptive and backward-looking. Acai's "specs describe how your system SHOULD behave" is prescriptive and forward-looking. That's not just a wording difference; it determines whether your spec leads or follows the code. In an agent-driven world where code is regenerated constantly, a spec that follows the code is useless. Acai gets this right.

The ACID concept is genuinely novel in this space. [[Spec-Driven Development]]'s Plumb tool extracts decisions from diffs; [[OpenSpec]] tracks spec deltas. But neither creates a stable, greppable ID that threads from requirement through implementation to test. ACIDs make the spec-code relationship mechanical rather than conceptual. That's a real contribution.

The weakness is the stable numbering constraint. The author acknowledges it: "you must re-align the code whenever your spec changes." In a fast-moving project, spec churn means constant renumbering and comment updates across the codebase. The `deprecated` and `replaced_by` flags help, but this is the kind of friction that kills adoption. [[Cognitive Debt]] warns about exactly this: when maintenance overhead exceeds comprehension benefit, developers route around it.

The comparison section is entertainingly biased ("I suffer from Not Invented Here syndrome") but contains a real insight about the descriptive-vs-prescriptive split with OpenSpec. The dismissals of SpecKit and Kiro are less useful -- they read like competitive positioning rather than analysis.

What's missing: any evidence of team adoption beyond the author. Every spec-driven tool in this wiki ([[OpenSpec]], [[Spec-Driven Development]], [[Spec-First Development at Benchling]]) faces the same adoption question. The author has built a solo practitioner's tool and extrapolated to team workflows. The dashboard's collaboration features (comments, accept/reject) are promising but unproven.

The "AI psychosis" framing is the most relatable part. The cycle of building meta-tools, realizing you're not shipping product, throwing it all away, and starting simpler -- that's the actual practitioner experience that most spec-driven manifestos skip. It makes the tool recommendation credible precisely because the author admits to the failure mode first.

## Connections

- [[Specifications as the Product]] — Acai is a concrete implementation of the "specs are the durable artifact" thesis
- [[Spec-Driven Development]] — The triangle model (spec-test-code); Acai adds stable IDs as the glue
- [[OpenSpec]] — Direct competitor; prescriptive vs. descriptive philosophy split
- [[Spec-First Development at Benchling]] — Enterprise version of the same insight: define once, consume generically
- [[The Dark Factory is a DOT File]] — The "dark factory" the author references; Acai is the spec layer that feeds it
- [[Write Only Code]] — If nobody reads the code, acceptance coverage replaces code coverage
- [[Cognitive Debt]] — The risk: ACID maintenance overhead could itself become cognitive debt
- [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] — Acai targets Level 3-4; the testmaxxing/factory pipeline aims at Level 5
- [[Feedback Loop is All You Need]] — ACIDs are a mechanical feedback loop between spec and implementation
- [[Agent Coding Workflow]] — Acai fits the "structured spec" end of the maturity spectrum

---
*Sources: [[summary/specsmaxxing]]*
*Last updated: 2026-05-14*
