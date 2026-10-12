# Anatomy — a Skill for Claude Code

One-paragraph precis: Ryan's Anatomy skill makes Claude Code invent a mechanical object — a marble run, a fruit machine, a water tank — whose moving parts *are* the concept being explained, drawn as a live isometric SVG figure with true numbers read from a shared model. The write-up is both a skill showcase and a worked example of skill design: one governing rule, a fixed pipeline, shipped examples, budgeted sizes, and honest timing data.

---

## What it is

Name an idea in one line; Claude invents the machine that does it. A hash map collision is a sorting desk filing letters into pigeonholes. An LRU cache is a bookcase. A bloom filter is a fruit machine that pays out only when all three reels (hash functions) have the word's flags up. The rule at the top of the skill's instructions:

> The object is the explanation.

The failure modes the rule exists to kill are "too abstract" (floating labelled boxes) and "too flat" (one slab). The cure is a recognisable machine of many small parts, each doing something — never copied from a reference picture.

## The pipeline

Ten fixed steps, with most quality decided in the first four, before any geometry:

1. Size it — Figure (40–80 parts, 30–90 min), Hero (80–150, 2–4h), Epic (never started without you)
2. Truth first — a small model that both the motion and the readouts read, so they can't disagree
3. Invent the object, one-to-one with the idea
4. List at least forty parts, grouped into base/structure/mechanism/controls/one lit subject

Then geometry is built once and painted back to front, motion is calm and time-based, and verification is "like a critic": screenshots, contact sheets, crops at 4×, automated checks, "until nothing is left to fix."

## The prompt is the idea, not the drawing

You almost never say what to draw — "Explain how an LRU cache evicts entries" is the whole prompt, and choosing the bookcase is Claude's job. Three words change the output: `interactive`, `WebGL` (shaders inside the isometric camera), `3D` (hours, not minutes). The skill's description is written so Claude reaches for it unprompted on any explanatory-diagram request.

## Honest numbers

- Watch movement: 1h54m, one pass, checked by the same agent
- Raptor 2 engine: 38 minutes to first working version, then ten hours to *right* — with an audit proving 6,320 overlapping pairs across 541 solids all draw in the right order
- Turbofan: 14h, five review-and-fix rounds

The pattern: "A first version that runs comes quickly. Most of the time goes into *the second look*, at four times the size, where the faults are."

## Distribution

Every figure is also a React component (`anatomy-figures`, five themes, 26 CSS tokens, per-figure imports) or a custom element. Figures pause off-screen, redraw only what moves, cap shader DPR at 1.5, and announce readouts to screen readers.

## Limits, stated plainly

Claude is "very good at building these and not always good at *seeing* them" — a part can be right in every number and still look wrong, and the best fixes came from a human watching the figure play and describing what felt off. References give mood, not objects. 3D is a beta. First figure = draft.

---

## Analysis

The most interesting thing here isn't the pretty figures; it's that the skill is a masterclass in skill *design*. One inviolable rule at the top. A numbered pipeline where quality is front-loaded into modelling before geometry. Six complete examples shipped with the skill "from a dial gauge of about 300 lines to the Raptor engine," so Claude opens the closest one before starting. Budget tiers with an explicit rule that the biggest size requires human sign-off. This is the same philosophy as [[Claude Code Skills System]] — SKILL.md as invocable expertise — but pushed further than most: the rule "the object is the explanation" is doing the work a page of prompt hints never could.

The timing table is the most valuable part for anyone building with agents. Everyone quotes the 38-minute first working version; the actual costs are 10–14 hours of review-and-fix rounds on the big machines. It's a concrete instance of the verification-bottleneck argument in [[Team-Wide Agentic Harness]] and [[The New Software Lifecycle]]: generation is cheap, the second look is where the hours go — and here even the "second look" is largely automated (audits, contact sheets, 4× crops) with humans supplying only taste-level feedback ("say what feels off").

There's also a quiet claim about what agents are *for*. The skill deliberately refuses boxes-and-arrows because, in the author's framing, a diagram tells you parts exist but makes the reader imagine the motion. Making Claude build a working mechanism instead is a bet that generation is now cheap enough to afford *more* explanation, not less — the disposable-code economics argument, applied to pedagogy rather than software.

Key themes: #tool #pattern #concept

## Related pages

- [[Claude Code Skills System]] — strengthens it: this is a worked example of the skills architecture doing something a prompt could never do, with the same name+description discovery model.
- [[Make Pages Interactive]] — a sibling Claude Code skill with a different target (annotating existing HTML vs. inventing explanatory machines); together they map the "Claude as explainer" niche.
- [[Team-Wide Agentic Harness]] — nuances it: Anatomy ships its examples *inside* the skill and its pipeline as reviewed instructions, a single-author precursor to treating harnesses as shared infrastructure.
- [[The New Software Lifecycle]] — complicates it: the 38-minutes-vs-14-hours numbers are a vivid data point for implementation compression while verification stays the bottleneck.

---
*Sources: [[raw/anatomy]], [[summary/anatomy]]*
*Last updated: 2026-10-13*
