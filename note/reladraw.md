# reladraw — Diagrams Where You Say Where Things Go

reladraw is a text diagram language built on an inversion of the usual deal: instead of declaring boxes and letting a layout engine decide where they land, the author states the arrangement in relative terms (`right of app`, `level with queue`, `between web and worker`) and the tool computes only the distances. It renders standalone SVG with zero runtime dependencies, ships as a CLI and a web component, and — the reason it earns a place in this wiki — is explicitly designed and documented for use by coding agents, complete with an installable skill. v0.16.0, ~11.6K lines of hand-rolled TypeScript, Apache-2.0, by Joe Walsh.

---

## Architecture

The pipeline is `compile()` in `src/index.ts`: parse → resolve → render, with errors blamed back to the correct source line even across multi-line statements.

- **`src/lexer.ts` / `src/parser.ts`** (1502 lines) — the largest module. A statement starts at the beginning of a line and continues onto indented lines, but indentation is deliberately *semantically empty*: it continues a statement and never nests anything ("there are no blocks"). Attributes and placements can appear in any order because a token ending in a colon unambiguously opens an attribute.
- **`src/constrain.ts`** — the conceptual heart, only 129 lines. Every placement compiles to one shape of fact: `pos[to] >= pos[from] + weight`. `fix()` makes an exact distance out of two opposing inequalities. `tightest()` solves the whole system per axis via longest-path from a virtual source — Bellman-Ford with the comparison flipped — so the solution is the *unique tightest arrangement*, found by arithmetic, never by search. Unsatisfiable loops are detected after N passes and walked backwards to report the exact placements that fight.
- **`src/resolve.ts`** (2873 lines) — three passes: build the containment tree, size every node bottom-up from its text, then convert placements to constraints and solve. Non-overlap is a fixed-point loop: solve, add separation constraints for collisions, re-solve. `reachability()` (Floyd-Warshall on the constraint graph) decides *which node may move* to separate an overlapping pair — the file must order the pair on some axis, and if nothing does, the tool refuses rather than guessing.
- **`src/search.ts`** — edge routing. Every box is a wall; a line may turn only where tracks cross (tracks sit a little way outside each wall, down the middle of each gap, and through the ends), reducing open space to a few hundred grid points. Dijkstra with cost = length + per-turn charge + charge for crossing already-placed lines. On a grid, staircases tie with single bends, so the turn charge is what picks the line a human would draw; remaining ties go over-the-top and round-the-right by convention.
- **`src/render.ts`** (3593 lines) — SVG emission, including curved routes that "sweep" through corners, lane-nesting for parallel edges, and themes.
- **`src/element.ts`** — the only browser-running module: a `<reladraw-diagram>` custom element that dedents its text content and replaces itself with SVG.
- **`tools/stress.mjs`** — a performance stress harness with a recorded `stress-history.tsv`, standing in for a conventional test suite.

## Key techniques

- **Gaps are minimums, not distances.** `right of b  left of c` starts the two targets one gap apart; wedging a node between them pushes them apart by exactly the wedge's width plus two gaps, and deleting it closes them back up. No coordinate ever goes stale when a text grows. This one rule does the work that manual nudging does in draw.io.
- **Ambiguity is an error, not a choice.** Two placements on one axis with nothing on the other is refused, naming the axis. Two nodes that could overlap with no stated order on either axis is refused, naming the pair. When the file *does* imply an order on both axes, separation happens along the axis of least overlap — the smallest movement — because that is what a person would do.
- **Derived, never stated, decoration.** Multiple edges between the same pair of sides take nested lanes; lane order is read off the solved layout (reciprocal edges read as a circulation) and only falls back to written order when nothing implies it. Edge texts bid into the gap they cross, widening it by exactly the text's need.
- **Semantics-over-pictures naming.** Shapes are named for meaning (`document`, not `folded-corner`) so a redraw never changes a diagram's sense; the lone exception is `circle`, which has no single meaning. `badge:` is defined as pure shorthand for a child node drawn as an icon placed beside its parent's text — one rule instead of many.
- **Agent-oriented error culture.** Errors quote the author's own words ("`edge a -> b: cannot leave a's right side, b is against it`"), and the shipped skill is candid that re-reading source confirms *intent* but not *outcome* — text overflow and line crossings are not verifiable from source yet.

## Design decisions

The trade is explicit and refreshing: reladraw sacrifices the "just declare the graph" convenience of Mermaid for predictability. Where Mermaid hides arrangement behind heuristics, reladraw makes arrangement the *source of truth* and distances the derived output. The cost is verbosity — every node needs its relationships spelled out — and a solver that refuses the file when the author underdetermines the picture, which is friction a casual user feels as pedantry.

Optimization-for-readability shows in odd places: `//` replaced `#` as the comment marker solely so `fill: #14532d` could be written the way every other tool writes a color; `/` as a line break requires whitespace on both sides so `TCP/IP` and URLs survive intact. The icons are path data baked into the tool rather than a font or linked file, because the output must remain a standalone SVG. Contributing policy: issues welcome, PRs not accepted — a single-author design with opinions held firmly.

## Comparison notes

- [[ascdraw]] occupies the adjacent niche of text-to-diagram; reladraw differs by making relative placement the primitive rather than auto-layout, so the source fully determines the picture.
- [[Automatic Layout of Railroad Diagrams]] and [[grok-mermaid — Terminal Mermaid Renderer via WebAssembly]] cover the auto-layout tradition reladraw positions itself against: there the arrangement is an output you cannot predict; here it is an input.
- [[Diagram Design]] and [[Common Diagram Mistakes]] discuss what makes diagrams readable; reladraw encodes several of those judgments into the tool itself (lanes, turn charges, separation along the least-overlap axis) rather than leaving them to the author.
- [[Load-Bearing Assumptions]] argues skills are load-bearing infrastructure; reladraw is a clean example — `npx skills add reladraw/reladraw` ships the skill in-repo (`.claude/skills/reladraw/SKILL.md`) and installs across Claude Code, Codex, Cursor and Copilot, with the skill's framing ("you cannot see the SVG you produced") written specifically for an agent's verification constraints.

---

*Sources: [[raw/reladraw]], [[summary/reladraw]]*
*Last updated: 2026-10-10*
