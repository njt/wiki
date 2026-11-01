# SLCR Launches

Wack's 2026-10-29 launch of SLCR (Sliceable, Linked, Checkable Requirements) — an open format for requirements written to be consumed by both humans and coding agents, built because "our own agents kept drifting from the spec over time." The deliverables at launch: a CLI tutorial, an agent skill for reading/writing the format, and an open-source repo.

---

SLCR is a bet that the durable artifact in agentic development is the requirement, not the code. The format's three adjectives name its claims: requirements should be **sliceable** (independent units agents can implement without reading everything), **linked** (connected to each other and, implicitly, to the code and tests that satisfy them), and **checkable** (verifiable against what an agent produced, not just readable). The announcement is deliberately modest — three short paragraphs, no benchmarks — which is honest for a format launch: a format's value is proven by adoption, not by a launch post.

---

> "we built SLCR because our own agents kept drifting from the spec over time"

The interesting word is *over time*. This is not the cold-start problem of a bad initial prompt; it is the slow divergence between a spec and a codebase as agents make change after change. That places SLCR in the drift-detection camp — closer to verification and review gates than to spec-writing methodology.

> "If you work on a long-lived codebase and you're handing more of the implementation to agents, SLCR gives you a way to check that what comes back still matches what you asked for."

Note the frame: *check that what comes back matches*, not *write better prompts*. The spec is positioned as an assertion the code must be validated against — requirements as the product, code as the regenerable output.

> "the agent skill teaches your agents to read and write SLCR"

Shipping the agent skill alongside the format is the load-bearing decision. A requirements format agents can't natively read is dead on arrival; Wack treats agent literacy as part of the launch, the same move Spec Kit made with its per-agent command installation.

## Analysis

SLCR enters a crowded and fast-consolidating space. GitHub's [[Spec Kit]] owns the workflow-toolkit niche, spec-first.dev owns the slash-command demo niche, and FizzBee's argument in [[Your .md File Is Not a Specification]] says prose spec formats — however linked and sliceable — hide untestable invariants that only formal analysis surfaces. SLCR's differentiator, if it holds up, is the *checkable* part being a first-class property of the format rather than a separate tool's job: the announcement never says how checking works (tests? model-checked properties? diff against acceptance criteria?), and that's the whole ballgame. A requirements format whose checking story turns out to be "run your existing tests" is a naming exercise.

The second open question is whether "sliceable, linked" survives contact with real spec maintenance. Linked requirements are a graph, and graphs need curation; the wiki's record on knowledge-graph-for-codebase tools ([[lat.md]], [[Understand-Anything]]) is that drift in the artifact itself is the recurring failure mode. SLCR is a solution to spec drift that introduces a new artifact — the spec graph — which will itself drift.

Still, the underlying diagnosis is right and widely corroborated: as agents do more of the implementation, the bottleneck moves to knowing whether the output is what you wanted. SLCR is one more entry in the verification-over-generation turn, and being an open format with an agent skill gives it a plausible adoption path that most bespoke spec formats never get.

**Key themes:** #concept (requirements as checkable artifact) #tool (SLCR CLI + agent skill) #pattern (verification over generation) #project (Wack's open format)

Related pages: this launch strengthens the case in [[The Coming Need for Formal Specification]] that informal specs can't anchor agentic verification, nuances [[Spec Kit]] — which orchestrates the spec-to-code workflow but leaves requirements as markdown prose rather than a structured, checkable format — and complicates [[Your .md File Is Not a Specification]], which argues only formal notations are truly checkable, by betting that a structured-but-informal format can carry enough of the load. It also gives [[When Spec-Driven Development Pays Off]]'s finding (governance pays on hard, multi-constraint work) a candidate mechanism: per-requirement checkability is exactly what makes sliceable governance cheap enough to apply selectively.

---
*Sources: [[raw/announcements]], [[summary/announcements]]*
*Last updated: 2026-11-02*
