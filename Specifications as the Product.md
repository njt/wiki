# Specifications as the Product

The economics of software have inverted. Code is now the cheap, disposable, regenerable artifact. Specs -- the documents that capture intent, constraints, and design decisions -- are the durable product. This isn't a prediction; it's already how the most productive teams work. When [[The Dark Factory is a DOT File]] says "software is cheap now, specs are the expensive part," it's naming a shift that [[Spec-Driven Development]], [[OpenSpec]], and [[Spec-First Development at Benchling]] all confirm independently. The pipeline DOT file is reusable; the factory code is dorodango -- polish it, throw it away, rebuild from spec.

---

## The Landscape

### The Spec-Code Inversion

The traditional model: specs flow into code, code is the deliverable. The new model: specs, tests, and code form a triangle ([[Spec-Driven Development]]), and the spec is the vertex with the longest lifespan. Code can be regenerated from a good spec in minutes. A spec cannot be reverse-engineered from sloppy code at any cost.

[[Simplicity in the Age of AI-Assisted]] explains the economics: "LLMs don't change what good programming looks like, but they change our relationship to simplicity." The real unlock isn't faster generation -- it's cheap rebuilds without inherited complexity. If the code is disposable, invest in the spec.

[[The Claude C Compiler]] confirms from the compiler domain: agents implement known abstractions well but invent nothing. The scarce resource shifts from writing code to deciding what deserves building. Lattner's observation: "well-documented systems gain dramatic advantage" as AI amplifies structure.

### Spec Formats and Tools

**Narrative specs.** [[How to Write a Good Spec for Agents]] defines five principles: start with vision, structure like a PRD, break into modular prompts, build in self-checks, include domain knowledge. The three-tier boundary system (always do / ask first / never do) is immediately implementable. The key finding: model performance drops as instruction count increases -- modular specs beat monolithic prompts.

**Living specs with deltas.** [[OpenSpec]] introduces the spec delta -- a diff showing how requirements evolved, not just how code changed. Reviewers see the change in intent. This directly addresses [[Zero Alignment]]'s coordination problem: if the spec delta is clear, review doesn't require reading all the code.

**Pipeline specs.** [[The Dark Factory is a DOT File]] argues for workflow specs in Graphviz DOT format -- human-readable, version-controllable, familiar. Each node declares whether it needs an LLM, a human gate, or a verification command. The pipeline is the blueprint; the runner is disposable.

**Structured YAML specs with stable IDs.** [[Specsmaxxing]] introduces `feature.yaml` -- YAML-formatted specs where each requirement gets a stable Acceptance Criteria ID (ACID) that threads through code comments and test annotations. The key innovation over markdown specs: requirements are individually addressable and greppable. The prescriptive philosophy ("specs describe how the system SHOULD behave") directly opposes [[OpenSpec]]'s descriptive approach and creates a bidirectional traceability matrix between spec and implementation.

**Schema specs.** [[Spec-First Development at Benchling]] shows the enterprise version: define each object once through a unified contract, and platform capabilities consume the schema generically. Adding a new object means one schema declaration, not N integration adapters.

**Deployment specs.** [[Verbose Deployment]] implements a 10-phase deployment pipeline as composable Claude Code skills, each independently useful. The pipeline adapts to your project -- it detects your stack rather than assuming one.

### Spec-Adjacent Tools

[[Trycycle]] puts specs through plan-strengthen-review loops with fresh agents at every stage. The key innovation: fresh eyes prevent stale context from accumulating. Up to 5 planning rounds and 8 review rounds -- heavy on tokens, thorough on quality.

[[Prefix Effects]] reveals that specs need to include naming conventions, because function names act as "alignment surfaces" that shape all subsequent agent-generated code. `secure_create_user` begets bcrypt; `energetic_create_user` begets asyncio. The first code committed to a repo is disproportionately important.

[[Write Only Code]] pushes the argument to its logical conclusion: if nobody reads the code, the spec is the only human-readable artifact. The Slop Radius metric -- how far unintended behavior can impact before detection -- becomes the critical measure.

## Key Tensions

**Living documents vs. stable references.** Specs that evolve with the code ([[Spec-Driven Development]]'s triangle, [[OpenSpec]]'s deltas) risk becoming as unstable as the code itself. At what point is the spec updated so often that it's not a reference anymore? [[How to Write a Good Spec for Agents]] acknowledges this tension but doesn't resolve it.

**Spec granularity.** [[How to Write a Good Spec for Agents]] says modular, per-domain spec files. [[The Dark Factory is a DOT File]] says one pipeline DOT file. [[Spec-First Development at Benchling]] says one unified contract. The right granularity depends on the problem, but there's no framework for choosing.

**Human specs vs. machine specs.** Specs written for human developers emphasize narrative and rationale. Specs written for agents emphasize structure and constraints. [[OpenSpec]] tries to serve both with spec deltas (human-readable intent changes) backed by structured requirement tracking. Whether this dual audience can be served by one document format is an open question.

**Adoption friction.** [[OpenSpec]] itself warns: "Specs only work if you actually read them." The history of software documentation is littered with tools that made documentation easy but couldn't make anyone care. The [[Compound Engineering]] 50/50 rule (half your time on system improvement) is the cultural prescription, but it requires organizational commitment that most teams lack.

## What's Missing

**Spec testing.** We have tools for testing code against specs ([[Trycycle]], [[Verbose Deployment]]) but no tools for testing specs against themselves -- checking for internal contradictions, missing edge cases, or ambiguous requirements before any code is generated.

**Spec evolution tracking.** [[OpenSpec]] has spec deltas, but nobody has built the git-blame equivalent for specs: who changed this requirement, when, and why? Linking spec changes to the outcomes they produced would close the feedback loop.

**Cross-repo spec management.** All current tools assume specs live within a single repository. For organizations with microservices, the spec coordination problem across repos is unsolved. [[speedrift-ecosystem]] hints at cross-repo coordination but doesn't address spec synchronization specifically. [[Specsmaxxing]] explicitly supports cross-repo tracking via its dashboard -- a feature spec can track implementations across frontend, backend, and microservice repos -- but this is unproven at team scale.

## Key Themes

#spec-driven #disposable-code #intent-review #naming-as-alignment #pipeline-specs

---
*Synthesis of: [[Spec-Driven Development]], [[OpenSpec]], [[Specsmaxxing]], [[The Dark Factory is a DOT File]], [[Spec-First Development at Benchling]], [[How to Write a Good Spec for Agents]], [[Trycycle]], [[Verbose Deployment]], [[Prefix Effects]], [[Write Only Code]], [[Simplicity in the Age of AI-Assisted]], [[The Claude C Compiler]]*
*Last updated: 2026-05-14*
