# Harth — AI Workspace for Architects

Harth (harth.build) is a browser-based AI workspace for architecture teams: it ingests real 3D building data from Revit/Archicad/SketchUp/IFC, puts every frontier text model and every image model on one spatial canvas, and pitches "purpose-built agents" that do the boilerplate while the architect stays on the design. The interesting thing here is not any single feature — it's the shape of the pitch: the agent-workspace pattern (model aggregation, agent sequencing, canvas-native feedback, shareable deliverables) packaged for a profession that isn't software engineering, positioned as a layer *alongside* the existing CAD tools rather than a replacement for them.

---

## What the page actually claims

Three product pillars, all stated as marketing copy with no technical detail behind them:

1. **Model aggregation.** "Every frontier model an architecture team could ever need" — Gemini, ChatGPT, Claude, Grok, Kimi — "switch between models without leaving the canvas," and likewise for image models: Nano Banana Pro, ChatGPT, Seedance, Flux, Qwen, "built-in, no extra subscriptions or logins required." This is the aggregator economics play: one seat, all labs, model choice as a setting rather than a procurement decision.

2. **Purpose-built agents.** "Purpose-built agents can handle your complex architectural work." "Sequence multiple agents at once to handle the boilerplate while you stay on the design." And deep research on new sites yielding "a substantiated report you can trust" — a claim doing a lot of work with zero substantiation of its own.

3. **A spatial workspace with real data.** Import from Revit/Archicad/SketchUp/IFC, 3D satellite terrain, neighboring buildings, comments "over the 3D, not outside it," image-to-3D, multiplayer cursors, client share links ("decide exactly what your client sees"), one-click export to PDF/image/3D (video "soon").

The strategic sentence is: "Harth works alongside your CAD, not instead of it. Your CAD stays where the building becomes precise." The FAQ repeats it: keep your existing CAD, "Harth takes one export and handles everything from there."

## Key quotes

> "Deliverables that took a week are done in an afternoon with AI agents."

The classic vertical-agent promise — and note what it picks as the agent's domain: *deliverables*, i.e. the communicative output around the design (boards, renders, reports), not the design itself. That's the same division of labor the coding-agent world landed on: agents take the volume work, the human keeps judgment.

> "Your CAD stays where the building becomes precise."

A quietly excellent positioning line, and an admission: precision stays in the incumbent tool. Harth claims the ideation, visualization, and client-facing layers — the parts where "minutes, not days" rendering and iterative visual judgment pay off — and explicitly does not claim the dimensionally-exact model. This is the "agent as satellite, system of record stays put" pattern.

> "A substantiated report you can trust."

The weakest claim on the page, and worth flagging precisely because it's unverifiable marketing copy. [[A Serious AI Product]] argues a serious AI product is built around verification; this page asserts trustworthiness rather than demonstrating any mechanism for it. For an agent doing site research that feeds real design decisions, that gap matters.

## Themes

- #tool — a vertical agent workspace product
- #pattern — model aggregation + purpose-built agents wrapped around a profession's existing toolchain
- #concept — coexistence positioning: agent layer alongside, not instead of, the system of record

## Analysis

The genuinely notable thing about Harth is what it says about where agent products are heading. The coding-agent wiki's core patterns — multi-model aggregation on one canvas ([[cmux]], [[Agent Flywheel]]), agents sequenced on boilerplate while humans hold the design judgment ([[Optimizing for Decision Points]]), feedback anchored on the artifact itself ([[Agentation]]'s "comments become structured context"), shareable deliverable links as the output boundary — all reappear here, transplanted to architecture. That's convergent evolution, and it suggests these are becoming generic product templates rather than coding-specific insights.

The other notable move is the coexistence stance. Most agent products pitch replacement; Harth explicitly does not — "no. Keep using Revit, ArchiCAD, Vectorworks." It readss as honest, but also as a strategic hedge: the 3D model where "the building becomes precise" is the part where wrong output is expensive and verifiable, and it's exactly the part Harth leaves alone. Compare [[Building Shippy — Agent Architecture for High-Stakes Domains]], where the architecture deliberately wraps a messy domain with deterministic verification; Harth's marketing gestures at the same boundary but never explains its mechanism.

Big caveats: this is a landing page, not a field report. There is no published information about how the "purpose-built agents" work, what grounding they have against the building data, or how "substantiated" reports are actually checked. Treat it as evidence of the vertical-agent-workspace pattern arriving in AEC (architecture, engineering, construction), not as a technical source about how to build one.

## Related pages

- [[Building Shippy — Agent Architecture for High-Stakes Domains]] — the serious-engineering counterpart: where Shippy wraps a high-stakes domain with deterministic CLIs and live-data evaluation, Harth's marketing asserts trust ("a substantiated report you can trust") without showing the mechanism; this source shows the demand, Shippy shows what meeting it actually takes.
- [[A Serious AI Product]] — Glyph's verification-first specification is exactly the standard Harth's unsubstantiated-report claim should be measured against; the page is a live example of the trust-without-verification marketing he's arguing against.
- [[cmux]] — the same multi-model aggregation insight on a developer canvas; Harth shows the pattern generalizing beyond coding to a spatial profession.
- [[Agentation]] — Agentation turns UI annotations into agent context; Harth's "commentary over the 3D, not outside it" is the same feedback-anchored-on-the-artifact idea applied to buildings instead of interfaces.

---
*Sources: [[raw/harth-build]], [[summary/harth-build]]*
*Last updated: 2026-10-11*
