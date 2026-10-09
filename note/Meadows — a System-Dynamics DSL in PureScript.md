# Meadows — a System-Dynamics DSL in PureScript

Meadows is a small text language for stock-and-flow diagrams — the system-dynamics notation of Donella Meadows' *Thinking in Systems* — plus a browser playground that compiles a model into a graph, draws it as a force-directed diagram in the book's own visual idiom (boxes, faucets on pipes, clouds, thin information arcs), and simulates it with forward Euler. It matters here as a study in how little machinery a well-chosen DSL needs: identity-by-name, direction-as-data, and shape-driven simulation semantics produce a working modeling tool in roughly 7,800 lines across two languages.

---

## Architecture

Two halves meeting at a single JSON seam:

- **Compiler (PureScript, `src/`, ~900 lines)**: `Lexer.purs` (positioned tokens, `R(`/`B(` loop lexemes) → `Parser.purs` (495 lines, parsec-style combinators over a `State`-threading `ParserT`, producing a `Tree` of `NodeExpr`/`StockExpr`/`FaucetExpr`/`ArrowExpr`/`LoopExpr`) → `Evaluator.purs` (~460 lines), which resolves names to ids in a `registry` map, applies annotations, deduplicates links, detects formula cycles, assigns band groups, and serializes `{nodes, links}` via `Simple.JSON`.
- **Frontend (TypeScript + d3, `ui/`, ~6,900 lines)**: `app.ts` (3,297 lines) imports `../output/Main/index` and calls `interpreter.go(input)` — the browser consumes the *compiled* backend, never its source. `layout.ts` isolates the band/slot/branch force layout so `test/layout.mjs` can drive the exact same math headlessly; `simulate.ts` (631 lines, zero dependencies) is the engine; `chart.ts`, `playback.ts`, `loops.ts`, `highlight.ts` cover the rest.

`Expr.purs` is the elegance center: the formula AST is declared once, parameterized by what a reference is — `Expr Ref` (parser's unresolved mentions) vs `Expr String` (evaluator's id-resolved form) — with `refsWhere`/`traverseRefs` as the single fold/rebuild pair, so adding a formula form is one case per concern, not one per concern per copy.

## Key techniques

- **Identity by name.** A `registry` maps each name to the id minted on first mention; every later mention anywhere resolves to that node. A stock drawn on one line can be arrowed into from three other lines. Clouds are the deliberate exception — every `|` is anonymous and never merges. Links carry ids, not spellings ("identity, not spelling" is stated as a seam rule in three files).
- **Direction as data.** `Dir = Rightward | Leftward` means `->`/`<-` and `=>`/`<=` share one production and one evaluator case; every direction-dependent rule is written once and cannot drift.
- **First annotation wins, as a unit.** `annotate` refuses any write once a node carries a value, schedule, or formula — a valueless mention never erases, and a losing formula doesn't even resolve its references. One predicate guards all three annotation kinds.
- **Shape-driven simulation semantics.** A bare faucet (no number, no formula) is not an error: the simulator walks the information arrows into it. One valued dot at the end ⇒ **goal-seeking** (`rate = gain × discrepancy`, exponential approach, dashed goal line on the chart); the dot's value plus the faucet's own stock meeting in a relay dot ⇒ goal-seeking at default gain 1; level and constant arriving separately ⇒ **reinforcing** (`rate = factor × level`). Ambiguous webs fall back to constant-rate. The diagram *is* the semantics.
- **Time shifts as both feature and cycle-breaker.** `x(t - T)` is a ring buffer of `T/DT` samples primed to the input's t=0 value. Crucially, a shift reads its own buffer, never its input's current value — so `eagerRefIds` (cycle detection) refuses to descend into `EShift`, and `a: (a(t - 1) + 1)` is legal where `a: (a + 1)` is a model error. The dealership's oscillation exists *only* through the delay; the README's figure-33 walkthrough shows the two dashed/solid line pairs trailing their partners by exactly their delay.
- **Physical realism as policy, not per-model code.** Stock outflows are rationed by `min(1, level/demand)` each step, so levels never go negative and chained stocks conserve material; faucet rates clamp at 0 (a tap never runs backward); non-finite arithmetic reads as 0.
- **Ports as minted nodes.** An information arrow never touches a stock's body: the compiler mints a `port` node (with a `parent` field) pinned to the stock's edge, and `logicalEnd` makes a port stand for its stock during arrow deduplication — so formula-implied arrows and hand-drawn ones compose without doubled arcs.
- **Testability by construction.** Layout and simulation are extracted precisely so headless Node tests (type-stripped `.mjs`) run the identical code the browser runs — the README calls this "no drift." `test/golden.mjs` (1,272 lines) pins byte-exact graph JSON and positioned error messages.

## Design decisions

- **Deliberate omissions**: no smoothing primitive (perception reads *are* the pipeline shift; exponential approach falls out of goal-seeking faucets), no exponent-notation literals, `5.` rejected, juxtaposition of two names *joins* them rather than multiplying (`pi t` is one name). Each restriction keeps the surface syntax small and the error messages positioned (`Tokenization error: line 1, column 3`).
- **PureScript for a compiler in a JS world** is the boldest choice: total functions and a real parser-combinator library for the semantics, with the cost being a build-coupling footgun the repo's own CLAUDE.md warns about (browser refresh silently runs stale code unless you `spago build`).
- **Annotations-as-behavior over equations-everywhere**: the book's balancing loops need no algebra — draw the arrows, give the goal dot a number, and goal-seeking emerges. The trade-off is that the semantics are implicit in graph shape, so ambiguous webs silently fall back to a constant-rate reading rather than erroring.
- **The flows checkbox is an argument, not a feature**: rates aren't accumulations so they're hidden by default, but when a model contains a delay the gap between what happens and what the decision-maker *sees* is the whole story — so in delay models the checkbox overlays the shift's input/owner pairs instead of raw rates.

## Comparison notes

- Compared to [[Dippin (Language)]], which builds a full compiler pipeline with IR, LSP, and simulator for *agent workflows*, Meadows shows the minimal end of the same DSL impulse: one parse pass, one evaluator, one JSON seam — and the agent-relevance is only incidental (its own CLAUDE.md is written for Claude Code).
- Compared to [[Malloy]], which compiles a modeling language down to SQL with a two-phase compiler, Meadows compiles a modeling language down to a *simulation* — both take "the model text is the product" seriously, but Meadows' compiler is ~900 lines where Malloy's is 93K.
- [[Automatic Layout of Railroad Diagrams]] treats diagram layout as a compilation problem with formal passes; Meadows takes the opposite route — d3 forces with a compiler-assigned *band group* heuristic (connected components over flow links) — and gets the book's distinctive look from icon geometry (the faucet's 79% base-offset `aimY`) rather than layout theory.
- Against [[Diagram Design]]'s "generate 40 diagram types as HTML" skill approach, Meadows is the artisanal counterpoint: one notation, deeply implemented, where the rendering idiom (Meadows' own book figures) carries pedagogical weight the generic tool doesn't attempt.

#tool #project #dsl #systemdynamics #purescript #compilers #simulation

---
*Sources: [[raw/meadows]], [[summary/meadows]]*
*Last updated: 2026-10-10*
