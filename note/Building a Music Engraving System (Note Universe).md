# Building a Music Engraving System (Note Universe)

A solo pianist-developer's write-up of the two-year engine behind Note Universe: a music engraving pipeline (layout + rendering) powering a sight-reading drill app, hand-built for a year and then maintained entirely via Claude Code — with LLM-composed MusicXML now becoming a content source for the app itself.

---

The engineering core is a classic two-phase split: an **engraver** that turns musical data into `Renderable` objects (chords, beams, accidentals, rests — each with bounding boxes and coordinates), and a **dumb renderer** that just draws glyphs at specified positions. Engraving is expensive and runs on demand; rendering runs at 60fps and stays under 16ms thanks to coordinate-based culling of the unbounded stream of Renderables that "infinite" exercises produce. The author chose an imperative layout algorithm over the constraint-based "springs and rods" approach used by Dorico — simpler, and adequate for the feature set.

Two testing ideas stand out. First, **ASCII fixtures** for unit-testing the engraver in isolation: a chord engraves into a small ASCII-art staff, self-documenting and far less brittle than asserting exact coordinates. Second, **screenshot goldens as dual-purpose artifacts** — over 200 of them, serving both as regression tripwires and as visual documentation of what the engraver can do, with embedded metadata in the output.

The rules of the domain come from Elaine Gould's *Behind Bars*, which the author calls "one of the best technical books I've ever read" — 700+ pages of visually-illustrated engraving conventions with no ISO standard behind them. Violate them and you get "uncanny valley sheet music" that experienced musicians can feel is wrong without being able to say why. The post includes a spacing function that converts Gould's spacing rules into a power-law formula — longer notes get more room, but sub-linearly.

The AI arc is the quietly interesting part. For provenance, the author states the text is human-written while the last year of development ran entirely through Claude Code. More substantively, the engraver API was originally designed cleanly enough that admitting a MusicXML importer was a small refactor — and that importer exists because the author bet LLMs would eventually emit valid, musically interesting MusicXML. Claude Fable composed a waltz in G major; it renders and is playable. The design-for-testability and decoupled API weren't agent-preparation, but they turned out to be exactly what made the agent a content pipeline.

---

## Key Quotes

> "uncanny valley sheet music that just doesn't look quite right to an experienced musician"

The best argument for domain-rule fidelity in software: the failure mode isn't a crash, it's subtle wrongness that only experts perceive. This is why the testing strategy matters — you can't eyeball 200 engraving behaviors.

> "This approach isn't suitable for testing every single behavior, but provides a quick and easy way to test some core engraving paths in a way that isn't brittle"

Honest scoping of a testing technique. ASCII fixtures are golden tests with a human-readable diff, and the author knows precisely where they stop being useful — a rarer admission than the technique itself.

> "I figured that at some point, LLMs would be sophisticated enough that they could generate valid and perhaps even musically interesting MusicXML."

The bet that reframes the whole project: the importer isn't a feature, it's an interface between the engine and generative AI as a content source. The architecture decision (a general engraver API) made the bet cheap to place.

> "I'm mostly just calling this out for transparency, as people are understandably sensitive about human vs. AI provenance these days."

A year of Claude-Code-driven development on a hand-started codebase, disclosed in one sentence. The interesting implicit claim: a solid hand-built foundation plus a clean API is what made full-agent maintenance tractable — not the other way round.

---

## Key Themes

- **#concept Decoupled layout and rendering** — the Renderables contract lets the renderer know nothing about music. The same boundary later let an external MusicXML source plug in with minimal work; the decoupling paid for itself twice.
- **#pattern Dual-purpose golden tests** — screenshot goldens that document capabilities while catching regressions; ASCII fixtures that self-document core engraving paths. Testing as specification, not just verification.
- **#tool Bravura + SMuFL** — Steinberg's freely-licensed font with a coordinate scale aligned to notation (1 em = 4 staff spaces) eliminated per-glyph fudge factors. Standards and open fonts as invisible infrastructure.
- **#person Solo craft at two-year scale** — a domain book as the spec, a prototype thrown away, a personal pain point (sight reading) as the product. Small-scope, high-taste engineering.

---

## Opinionated Take

This is a craft essay wearing a music costume, and it's better for it. The provenance disclosure will draw the eyes, but the actual lesson is unfashionable: the agent era went smoothly here *because* the first year was done well by hand. The engraver API was "reasonably solid," the renderer was isolated, the engraver was unit-testable — so handing development to Claude Code was a low-risk swap, and adding an LLM-generated content pipeline was a two-step change. Compare that to the usual dark-factory story where agents generate the mess they then have to maintain.

The MusicXML bet is the sharpest idea in the post. Most "AI generates content" stories are about generating text into a database; here the generated artifact has to survive a 700-page rulebook and a pixel-accurate renderer, which makes the LLM's output checkable in a way prose never is. A wrong waltz is visibly wrong. That's a nicer verification loop than most agent pipelines have.

The ASCII fixtures deserve stealing outside music: any domain with a small visual/spatial grammar can probably be golden-tested as monospace art, and the self-documenting property means the test suite doubles as onboarding documentation.

---

Related pages in this wiki: this strengthens [[Building low-level software with only coding agents]] with the complementary case — there a library was agent-built from scratch, here a hand-built engine was handed to agents for a year, and both worked because the boundaries were clean. It nuances [[Building a RAG Pipeline for Semantic Code Search (JetBrains)]]-style verification discussions with a domain where output correctness is visually checkable by construction (screenshot goldens over 200 cases). It complicates [[How to Follow a Drummer]] and [[Klangio Transcription Studio]] by approaching music + software from the notation side rather than audio — Klangio solves "audio → notation," this engine solves "notation → pixels," and LLM-composed MusicXML sits between them as a new content source. And it resonates with [[What I learned building an opinionated and minimal coding agent]]-scale solo projects: one person, a domain book for a spec, and taste as the quality bar.

---
*Sources: [[raw/building-a-music-engraving-system]], [[summary/building-a-music-engraving-system]]*
*Last updated: 2026-11-09*
