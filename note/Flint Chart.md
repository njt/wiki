# Flint Chart

A visualization intermediate language (IL) that separates **what a chart means** from **how it's rendered**, letting AI agents produce polished, multi-backend charts from compact specifications. Built by Microsoft Research, the compiler resolves 70+ semantic types, fits content to canvas with physics-inspired layout, and emits Vega-Lite, ECharts, Chart.js, Plotly, or native Excel output from the same input. The thesis: declarative grammars like Vega-Lite get brittle when semantic meaning diverges from storage representation; Flint makes semantic types first-class objects and derives encoding, layout, and style from them.

Flint is the most architecturally ambitious visualization project in the AI-tools space — not a thin wrapper over existing libraries but a genuine compiler with a three-stage pipeline, a formal theme system, and a group-theoretic pivot model for view transformations. At ~78K lines of TypeScript with five complete backends, it represents a different bet than most agent tools: invest heavily in the intermediate representation so agents (and humans) can be sloppy about chart authoring and still get polished output.

#visualization #tool #project #agents #compiler

---

## Architecture

Flint is a **three-stage compiler pipeline** — backend-agnostic frontend and optimizer, then per-backend code generation. The canonical orchestration lives in `packages/flint-js/src/vegalite/assemble.ts`, but every `assemble*()` function follows the same path.

### Stage 1: Compiler Frontend (`core/resolve-semantics.ts`, 475 lines)

Maps raw data + semantic types + encoding bindings to `ChannelSemantics` — a flat, backend-agnostic record per channel. The resolver handles two layers:

**Field properties** (from `core/field-semantics.ts`, 1,122 lines, and `core/type-registry.ts`, 186 lines): Each semantic type registers its tier (T0 family → T1 category), visualization category (quantitative/ordinal/nominal/temporal/geographic), aggregation role (additive/intensive/signed-additive/dimension), domain shape (open/bounded/fixed/cyclic), diverging classification, format class, zero-baseline class, and domain padding. Types not in the registry fall back to value inspection via `inferVisCategory()`.

**Channel properties**: The same `YearMonth` field may be temporal on `x` in a line chart but categorical on `color` in a grouped bar. Channel semantics prevent year-month integers from being treated as quantitative magnitudes. The resolver also handles temporal format detection — analyzing date granularity (years → seconds) via a voting algorithm over 6 timestamp resolutions (`computeDataVotes()`), accounting for whether all values share the same year/month/day — and ordinal sort order (canonical month names, days, quarters).

**Temporal data conversion** (`convertTemporalData()`): Two-digit years expanded (0→49 = 2000s, 50→99 = 1900s), Unix timestamps detected and converted to ISO strings, Decades/Years normalised to prevent sub-year ticks.

### Stage 2: Optimizer (`core/compute-layout.ts`, 1,908 lines)

The optimizer receives `baseSize` (target) and an optional `canvasSize` (ceiling), then produces a `LayoutResult` that keeps the chart readable within the available space. Two classes of layout behavior:

- **Discrete axes (bars, heatmap cells)**: Elastic "spring" model — compress toward minimum readable step, stretch canvas if needed. Band step computed per-category with configurable padding fraction. Grouping support with lanes-per-band budgeting.
- **Continuous axes (scatter, line)**: "Gas pressure" model — stretch when mark density exceeds overlap budget. Configurable `markCrossSection` (σ) per axis, with series-count-based pressure as an alternative to pixel counting.

**Global optimization**: Aspect ratio banking (to 45° for connected marks), facet row/column wrapping, and non-Cartesian sizing (treemap, gauge, pie from component counts). Overflow is handled **gracefully**: when discrete cardinality exceeds the canvas budget, the optimizer filters data and attaches `ChartWarning` metadata instead of rendering an unreadable chart.

The `LayoutResult` is **target-agnostic** — abstract pixel dimensions (`subplotWidth`, `xStep`, `stepPadding`, facet grid) that each backend translates to its own coordinate system. The engine does not know about Vega-Lite encodings or ECharts grid objects.

### Stage 3: Code Generators

Each backend lives in its own directory under `packages/flint-js/src/` with a canonical structure: `assemble.ts` (orchestrator), `instantiate-spec.ts` (shared assembly logic), `colormap.ts` (backend-specific color mapping), and `templates/` (one file per chart type). Template counts by backend:

| Backend | Chart types | Notes |
|---------|-------------|-------|
| Vega-Lite | 25 | Most complete; themes supported |
| ECharts | 32 | Includes Sankey, Sunburst, Tree, Gauge, Funnel |
| Chart.js | 20 | Bubble, Radar, Rose, ECDF |
| Plotly | 38 | Includes Map, Choropleth, KPI Card, Sparkline, Regression, Violin |
| Excel | 18 | Native Office.js, worksheet matrix + chart type bindings |

Each chart type is a `ChartTemplateDef` with: display name, template skeleton, allowed channels, `markCognitiveChannel` (position/length/area — drives zero-baseline and compression), `declareLayoutMode()` (axis band flags, type overrides), and `instantiate()` (emit spec from context).

### MCP Server (`packages/flint-mcp/`)

The MCP server provides tools for listing chart types, validating inputs, compiling specs, and opening interactive previews. Rendering goes through headless backends (Vega-Lite → SVG via `vega`, ECharts → SVG via `echarts` SSR, Chart.js → PNG via `chartjs-node-canvas`). The server can read local JSON/CSV/TSV files referenced by path but does not fetch remote URLs.

---

## Key Techniques

### Semantic types as first-class compilation objects

This is Flint's defining idea and the thing that distinguishes it from every other charting library. The type registry (`core/type-registry.ts`) encodes orthogonal compilation dimensions for 70+ types:

```
Price → T0:Measure, T1:Amount, currency format, zeroBaseline:meaningful, aggRole:intensive
Temperature → T0:Measure, T1:Physical, unit-suffix, zeroBaseline:arbitrary, diverging:conditional
YearMonth → T0:Temporal, T1:DateGranule, visEncodings:[temporal,ordinal], domainShape:open
Profit → T0:Measure, T1:SignedMeasure, signed-additive, diverging:conditional, zeroBaseline:meaningful
Correlation → T0:Measure, T1:SignedMeasure, domainShape:bounded, diverging:inherent, zeroBaseline:meaningful
```

These dimensions cascade through the pipeline: `aggRole` determines whether stacking is semantically valid (additive measures can stack; intensive ones like Temperature cannot), `zeroBaseline` decides whether the axis starts at zero (meaningful for Count, arbitrary for Temperature), `diverging` controls whether the color scheme splits at a midpoint, `formatClass` drives axis/tooltip number formatting, and `domainShape` influences nice rounding.

The tiered type system (T0 → T1 → T2) allows graceful degradation: if an agent supplies a coarse label like "number", Flint inspects the data values and falls back to the best available inference rather than failing.

### Physics-inspired layout models

Rather than requiring authors to specify pixel dimensions, Flint models chart sizing as a physical system:

- **Spring model (elastic/banded)**: Discrete categories act like points connected by springs. Each category wants `defaultBandSize` pixels (configurable, scaled to canvas size). When N categories × band size exceeds canvas width, springs compress toward `minStep`. When it's below, bands stretch up to `maxBandSize`. The elasticity exponent (default 0.5) controls how aggressively the system compresses — below 1 means early compression is gentle and only dense extremes pack tightly.

- **Gas pressure model (continuous)**: Continuous marks (scatter points, line vertices) exert "pressure" proportional to density. The `markCrossSection` (σ) parameter represents each mark's footprint in pixels. When `N * σ / axisLength` exceeds an overlap threshold, the axis stretches to relieve pressure, bounded by `maxStretch`. The per-axis stretch computation: `stretchFactor = (N * σ / baseSize)^elasticity`, clamped to `maxStretch`.

- **Band aspect ratio correction**: When a banded axis faces a continuous one (bar chart), each band has a natural AR = continuousAxisSize / stepSize. If that exceeds `targetBandAR`, the continuous axis shrinks via log-space blend so bars don't become excessively tall or thin.

This is a fundamentally different approach from Vega-Lite's fixed-width step sizing or ECharts' container-fill behavior. Flint models constraints and lets physics solve for dimensions, which means the same chart spec adapts across canvas sizes and data cardinalities.

### Formal ThemeSpec — design languages separate from chart specs

The theme system (`core/theme/types.ts`, 818 lines of type definitions) is a three-level architecture:

**Level 1 (authored)**: `ThemeSpec` — a pure JSON document. The invariant: it never names a chart type, a positional channel, a mark type, a field, or a backend property. It only states policy about ink, type, structure, marks, labels, legends, and layout. Ten presets ship in `core/theme/presets/` (Economist, Swiss, Nature, NYT, McKinsey, Datawrapper, Power BI, Pop, Cartoon).

**Level 2 (grounded)**: `DesignDecisions` — the ThemeSpec bound to a specific chart. Every role is resolved against the actual axes, series count, category count, mark channel, and canvas size. This is where placement preferences become actual placements, where legend sizing meets actual series counts, and where data label feasibility is computed from available space.

**Level 3 (realized)**: Backend-native config — `vegalite/theme.ts` translates `DesignDecisions` into Vega-Lite config objects. Other backends accept the theme field but don't yet apply it.

Themes support **inheritance**: `{ extends: 'economist', ink: { series: { single: '#6b3fa0' } } }` merges nested objects and replaces scalar values. This means a brand team can define one file that inherits a house style and overrides only brand colors, then apply it across an entire chart library.

### Named View transformations — group-theoretic pivot model

The pivot system (`core/pivot.ts`, 992 lines) treats chart exploration as an algebraic problem. From one authored encoding assignment, four generators compute an orbit of alternative views:

| Generator | Symbol | Operation | Example |
|-----------|--------|-----------|---------|
| τ (transpose) | `flip:x-y` | Swap two axis slots wholesale | Vertical → horizontal bar |
| σ (permute) | `swap:y-color` | Exchange a position field with an auxiliary channel | Scatter: map measure from y to color |
| γ (shift) | `series:row` | Route a discrete series field among color/group/facet channels | Grouped → faceted |
| θ (transition) | `type:Strip Plot` | Re-render as a sibling chart type | Grouped Bar → Stacked Bar → Pyramid |

The orbit is **computed over Flint's backend-neutral encoding IR**, so the same state ids apply across all backends. Validity checks are typed: σ only swaps within the same field profile (measure↔measure, category↔category), while τ is profile-agnostic (flipping slots, not roles). Deduplication handles round-trips (Scatter → Strip → Scatter folds back to Default). Line charts omit τ so Flint never offers a vertical line chart.

A central registry (`core/chart-transitions.ts`, 166 lines) declares the θ graph — which chart types can transition to which siblings — with declarative gates like `requireOrderedAxis`, `requireNonNegative`, `maxCategoryCardinality`, and `requireNoSeries`. Edges are candidates only; runtime filtering checks actual data + backend availability.

### Dynamic templates with dual control systems

Chart templates are not static JSON — they're factory functions. Each `ChartTemplateDef` defines a `declareLayoutMode()` hook (runs before layout) and an `instantiate()` hook (runs after layout to produce the final spec). Templates can:
- Override encoding types (Q→O for bar chart axes)
- Set band/group sizing parameters
- Apply custom overflow strategies
- Normalize encodings before the pipeline runs (sparklines remap series to facets)
- Post-process specs after assembly

The dual control system separates concerns cleanly:
- **Category A (`ChartPropertyDef`)**: Visual properties (corner radius, opacity, curve type). The `check()` predicate gates applicability; templates define them; the pipeline evaluates them. Hosts render controls from the same descriptor.
- **Category B (`EncodingActionDef`)**: Encoding transforms (sort, color scheme, aggregate, orientation). These mutate the *input* so the full pipeline re-runs — essential because sort changes which categories survive overflow, and orientation changes which axis is banded.

---

## Design Decisions

### Optimized for agent authorship, not hand-crafted control

Flint's design optimizes for the primary user being an LLM agent, not a human data visualization expert. This shows in:
- **Compact specs**: A complete chart spec can be ~10 lines of JSON. The compiler infers everything else.
- **Semantic inference**: Agents supply `{ weight: 'Quantity' }` rather than configuring axis scales, zero decisions, formats, and color schemes individually.
- **Graceful degradation**: Tiered type resolution means coarse labels still produce reasonable charts.
- **Overflow as first-class**: Rather than requiring agents to predict cardinality, Flint handles overflow gracefully and reports warnings.
- **Pivot system**: Agents don't need to generate six different chart specs to explore views — one spec, four generators, and the orbit is computed.

The trade-off: a human visualization expert who wants precise control over every tick mark and color stop will find Flint's derived decisions occasionally wrong. Flint is not a replacement for hand-authored D3 — it's a different category of tool.

### Five backends, one abstraction — but not all equal

The five-backend strategy is ambitious and has clear gaps. Vega-Lite is the most complete (themes, all chart types, faceting). ECharts has an excellent template count (32 chart types) but no theme support. Plotly has the most templates (38) but self-manages facets for composite charts. Chart.js has the fewest (20). Excel is fundamentally different — it produces a native chart artifact, not a rendering spec.

This is honest: the architecture acknowledges the asymmetry rather than papering over it. New backends need only implement Stage 3. The frontend and optimizer remain unchanged.

### TypeScript-first, Python preview

The architecture runs entirely in TypeScript/Node.js. The Python port (`packages/flint-py/`) is source-only preview. This limits adoption in Python-dominant data science workflows but makes sense given the agent integration target: most coding agents (Claude Code, Codex, etc.) operate in TypeScript/JavaScript environments.

### Layout engine as backend-agnostic abstract pixels

The `LayoutResult` is backend-agnostic — it produces abstract pixel dimensions that each backend must translate. This is clean architecture but means each backend carries translation logic (ECharts computing `barWidth` from `step × (1 − stepPadding)`, Vega-Lite using native `width: { step: N }`, Chart.js filling the container). The translation is a recurring source of subtle bugs (test data fixtures in `shared/test-data/` reveal backend-specific edge cases in layout translation).

---

## Comparison Notes

Flint's closest cousin in the wiki is [[Malloy]] — both are semantic modeling languages that compile to backend-native output through an intermediate representation. Malloy separates data semantics from SQL generation; Flint separates data semantics from visualization generation. Both use a compiler architecture with an IR that decouples the authoring language from the target. Malloy's symmetric aggregates (correct aggregation across joins) is analogous to Flint's semantic types preventing incorrect operations (stacking non-additive measures, diverging color schemes on sequential data). Malloy uses a Vega renderer for visualization; Flint would be a natural visualization layer for Malloy's query output.

Unlike tools that wrap a single charting library (most agent visualization tools), Flint is a **multi-backend compiler** — one input, five outputs. This is architecturally bolder and more complex, but it makes Flint portable across rendering environments. An agent can produce Vega-Lite for web rendering, ECharts for enterprise dashboards, and Excel for business users, all from the same semantic spec.

Flint ships agent skills (`agent-skills/`) for chart and theme authoring, making it a concrete example of the [[Claude Code Skills System]] pattern: reusable expertise packaged as filesystem-native plugins. The skills guide agents through Flint's template catalog, semantic type selection, and theme application.

The theme system's insistence on never naming backend properties is architecturally similar to the separation in [[Igor Schwarzmann Design Systems]]: design tokens are abstract; realization is separate. Flint's ten presets (Economist, Swiss, Nature, etc.) demonstrate that a rich visual identity can be expressed without backend coupling.

Flint's compiler architecture — three stages, IR-based, backend-agnostic core — is the same pattern powering [[Kuna — Agent-First Decompiler]] (decompilation as compilation) and [[sem]] (semantic version control via tree-sitter IR). The compiler-as-architecture pattern is recurring across tools that need to map between abstraction levels.

[[TALA (Diagram Layout Engine)]] is the same instinct in the diagram domain: let the agent place nodes (its strength) and compile edge routing deterministically (its weakness). Flint and TALA converge on the principle that the *mechanical* half of visual output should never be the model's job.

---
*Sources: [[raw/flint-chart]], [[summary/flint-chart]]*
*Last updated: 2026-08-06*
