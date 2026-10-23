# Shop — a Design Canvas Built on Code (html.moldandyeast.com)

Shop is an artifact in the html.moldandyeast.com library of single-file, dependency-free HTML tools: a working design canvas where a person and AI agents co-design side by side and "every edit stays real, reviewable code." The source is a structural archive — llms.txt, index entry, and the app's own CSS/JS comments in file order — documenting a 696 KB HTML file that contains an entire design-tool stack: a grammar→IR→DAG→runtime core ("Trellis"), canvas-to-code round-tripping, a semantic UI language with a learned taste model, reviewable agent turns, and local-first project persistence.

---

## What it is

The library's premise: no prompts, just finished artifacts handed to an agent as a reference — "like this, but…". Shop (added 2026-10-21) is the flagship proof. Inside one file:

- **Trellis core** — a builder grammar where "N is the one builder node. Everything is a fluent chain," any value may be a lambda, and lambdas reading state stay live at runtime while the rest fold at build time.
- **Compile / Render / Export** — design → flat IR node table + DAG → live DOM ("the canvas is real HTML/CSS, like Paper"), exporting to Paper HTML, standalone HTML, JSX, IR JSON, or builder code.
- **Canvas edits → code** — canvas gestures write back as `edits({...})` entries keyed to stable layer paths, applied in rounds, with rename/move identity preserved ("'Row' never makes its sibling 'Row#2' answer to the first one's edits").
- **SUL + the Lab** — a semantic UI language that streams (every prefix is a valid program), styled by a 16-number learned taste vector θ, trained from A/B picks by a hand-wired ReGLU preference model; five bootstrap scorers, "their mean is the prediction, their disagreement is honest doubt."
- **Co-design** — one surface for every agent (window.shop API, Claude in the page, WebMCP): tiered reads, all-or-nothing code-mode writes, turns the person reviews and can reject without losing their own edits, pins anchored to layers, and undo/redo that keeps turns honest. A database bridge lets external Claude sessions post requests to `inbox/<id>` the page runs as reviewable steps.
- **Projects** — one file per design, local-first, mirrored to the artifact database, with an automatic restore point before every agent turn.

## Key quotes

> "A design canvas built on code, where you and AI agents design side by side and every edit stays real, reviewable code."

The pitch in one line: the canvas is not a parallel artifact that syncs to code — it *is* code, viewed. Every gesture lands in the program as an edits entry.

> "lambdas that read c.s (state) stay live at runtime; the rest are folded at build time"

A clean static/dynamic split: reactivity is opt-in by data dependency, so the IR/DAG stays deterministic where it can be.

> "not a sandbox, but agent code does not get the page's obvious doors (network, storage, capabilities, this API)"

Refreshingly honest framing of agent containment inside a single file — capability-stripping rather than security guarantees, stated as such.

> "their mean is the prediction, their disagreement is honest doubt"

The taste model's calibration rule: low scorer disagreement plus low confidence means "these are equally good," so the agent acts alone when sure and asks when not.

## Themes

#tool #concept #pattern

## Opinionated take

This is the most interesting artifact in the wiki's design-tooling cluster precisely because it is the anti-Paper: where [[Paper — the Connected Canvas for Teams Shipping with Agents]] is a funded company whose canvas *bridges* to code over MCP, Shop is a single file that refuses the bridge entirely — the program is the design, the canvas is a view, and agent edits are code edits with review gates. The comment archive shows the hard problems are not the ones you'd guess: they are identity and rename semantics for edits, focus preservation across re-renders, and undo/redo that keeps agent turns honest. The learned-taste model with scored doubt is a small but serious answer to the "give agents taste" problem the workflow essays keep raising. The limitation is legibility at scale: fluent chains and path-keyed edits work because the whole artifact is 696 KB a person can audit; whether the pattern survives a real codebase is exactly what the single-file library never has to answer.

## Related

This source strengthens [[Paper — the Connected Canvas for Teams Shipping with Agents]] — Shop explicitly targets Paper compatibility (HTML export, token namespaces, whole-pixel text semantics) while inverting its architecture from sync-between-artifacts to code-as-the-only-artifact. It nuances [[Design Systems for AI Agents (Luke Wroblewski)]] — tokens-as-CSS-variables in shared namespaces are the same steering-layer idea, but Shop makes the canvas itself consume them. It complements [[Make Pages Interactive]] — both are "hand the agent a finished HTML artifact as the reference" moves, with Shop adding the reviewable-turn co-design loop. And it complicates [[Agentation]] — both connect agent feedback to specific on-canvas elements, but Shop gives agents write access with reject-able turns rather than read-only annotation.

---
*Sources: [[raw/html-moldandyeast-com]], [[summary/html-moldandyeast-com]]*
*Last updated: 2026-10-24*
