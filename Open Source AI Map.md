# Open Source AI Map

A curated, hand-scored catalog of ~458 open-source AI products across 15 categories, published as an interactive maturity map at [map.currentai.org](https://map.currentai.org/). The project scores every product on three axes (openness, adoption, capability) backed by primary-source citations, then rolls up categories into a 0–5 maturity stage and a set of gaps that name what the open ecosystem still lacks. Its core insight: openness is orthogonal to maturity — a category can have strong products that simply aren't fully open, and that gap is the map's reason to exist.

---

## Architecture

The project is a **curated YAML data repository + deterministic build pipeline**, not a dynamic web app. The frontend lives in a separate repo. The pipeline is three stages: `validate` → `serialize` → `render`.

### Data model: four concerns + a manifest

All curated data lives in `sources/` as flat YAML files:

| Concern | Directory | Key fields |
|---------|-----------|------------|
| Organizations | `sources/organizations/` | `display_name`, `type` (company/lab/foundation/individual/government/unknown), `products:` roster |
| Categories | `sources/categories/` | `display_name`, `strapline`, `weights` (adopt/cap), ordered `products:` roster |
| Products | `sources/products/` | `display_name`, `type` (model/software/dataset/hardware), typed artifact URLs |
| Scores | `sources/scores/` | `openness` (class + 0-5 score), `adoption` (1-5 level), `capability` (1-5 score) — all with `sources:` citations |
| Taxonomy | `sources/taxonomy.yaml` | Three arcs (Columbia ontology layers), each with ordered category list |

The key structural choice: **products do not carry an `org:` field**. Organization membership is declared exclusively in the org file's `products:` roster, enforcing a single source of truth. The same pattern applies to categories: each product slug must appear in exactly one category roster. Cross-file invariants catch duplicates and orphans at validation time (`build/validate.py:50-98`).

### The three axes

- **Openness**: A categorical `class` (12-value vocabulary, type-dependent: `open_source`, `open_weights`, `open_core`, `source_available`, `restricted`, `gated`, `documented_only`, `closed`, `open`, `open_hardware`, `open_toolchain`, `documented`) plus a 0–5 `score` with a components breakdown. The class is the cross-category normalizer; the score is within-type only.
- **Adoption**: 1–5 measuring real usage (downloads, active users, deployments). GitHub stars capped at level 3. Five signal types: `active_users`, `usage_volume`, `reported_traction`, `stars_fallback`, `unknown`.
- **Capability**: 1–5, with a `basis` field (benchmark name, `feature_matrix`, or expert judgment). Comparable within category but not across.

### Stage and gap computation

The pipeline's centerpiece is `_stage_and_gaps()` in `build/serialize.py:86-141`. It places each of the 15 categories on a 0–5 maturity ladder by counting mature, fully-open products:

1. Collapse each product's `openness.class` into a 3-way bucket: **open**, **open-ish**, or **closed** (`_gap_bucket()`, `serialize.py:59-64`).
2. Compute each product's **maturity score**: a per-category weighted blend of adoption and capability, rounded to 2 decimal places to prevent float epsilon from deciding stage boundaries.
3. Only products in the `open` bucket with maturity ≥ 4.5 count as "mature" — a deliberately demanding bar.
4. Count mature open products: ≥4 → Stage 5, ≥1 → Stage 4, else bucket by best-open score (3.5, 3.0, 2.0 thresholds).
5. Derive gaps: `void` (no open options), `capability` (best open not capable enough), `adoption` (capable but under-adopted), `maturity` (not enough mature open products), `openness` (mature options exist but aren't fully open).

The openness gap is the system's signature contribution: it's **orthogonal** to maturity, not part of a linear scale. Open-weights models never advance a stage, no matter how capable.

### Renderer: code generation, not dynamic templating

`build/render.py` (1,122 lines) generates `notebooks/ai-stack-map.py` by:

- Reading `docs/methodology.md` (the hand-authored canonical methodology), substituting `{placeholders}` with live counts, converting Markdown to HTML via Python's `markdown` library
- Reading straplines and weights from source category YAML files at render time
- Dynamically generating one `@app.cell` function per category (15 cells)
- Injecting JavaScript interactivity (filter/sort controls, detail modals) via hidden iframe `onload` handlers — marimo strips `<script>` tags, but the iframe bootstrap survives static export

The rendered notebook is self-contained (data embedded as a JSON literal, no API calls), making it static-export-friendly.

### Validation: stronger than schema alone

`build/validate.py` (161 lines) enforces JSON Schema validation plus a web of cross-file invariants: roster completeness (exactly one category, exactly one org per product), taxonomy completeness (each category in exactly one arc), openness class validity per product type, the stars cap (`stars_fallback` cannot exceed level 3), source requirement (non-null scores must cite a source), and long-tail fixture synchronization (the frozen count must match the live product count).

## Key techniques

**Openness as orthogonal axis, not a linear spectrum.** The single most important architectural decision. A category can hold strong, widely-adopted products that aren't fully open, and the system explicitly reports this as an "openness gap" rather than downgrading maturity. This is the distinction the map exists to draw — and it's why `build/serialize.py:88` counts open-only for stage advancement.

**Float-epsilon-aware round-then-compare.** The maturity score undergoes `round(value, 2)` before comparison against stage thresholds. Otherwise `0.3 * 3 + 0.7 * 3 = 2.9999999999999996` could push a product into the wrong stage. The test suite validates this explicitly (`test_serialize.py:40-46`).

**Declared disclosure gap, not inferred.** The `disclosure` gap (frontier's training data recipe is invisible) is set with `disclosure_gap: true` on the category YAML. It's an editorial judgment about the *closed* world that open product scores cannot express. Making it declared prevents it from silently toggling when the roster changes. It can appear at any stage, including Stage 5 (`serialize.py:138-139`).

**Self-healing long-tail.** The frozen uncategorized sample is filtered at build time to remove products that now appear in the scored set. As analysts score more products, they automatically disappear from the uncategorized display without manual fixture cleanup (`serialize.py:169-177`).

**Source-citation-as-requirement, not scoring-as-formula.** Every non-null score must cite a primary source with URL, what-it-shows, and access date. The pipeline enforces this but does not compute scores — they are editorial judgments. This trades automation for auditability.

**Explicit policy parameters.** Every stage threshold (`_MATURE_MIN = 4.5`, `_STAGE5_MIN_MATURE = 4`, `_CAPABLE_MIN = 4`) is a named constant, described as "deliberate, tunable choices" rather than fixed law. The methodology treats the scoring model as a tunable instrument.

## Design decisions

**Auditability over automation.** Every score is human-judged with citation trail. This makes the map trustworthy but means coverage grows at analyst speed (~458 scored out of ~10K+ discovered, ~4.5% coverage).

**Openness purity over inclusiveness.** Only fully-open products advance maturity stages. Open-weights models (Llama, Mistral) never advance a stage. The methodology acknowledges this is "consequential but bounded" — only 3 of 15 categories would change if the rule were relaxed.

**Static snapshot over live dashboard.** The rendered map embeds all data and uses JS for interactivity. No API, no database queries in production. Fast and offline-capable, but reflects a point-in-time build.

**Single-source-of-truth over convenience.** The taxonomy manifest owns display order, arc grouping, and layer assignment. Category files don't have `arc` or `order` fields. The methodology Markdown is the canonical prose source. One edit propagates everywhere.

**Sacrificed: cross-category comparability.** The raw 0–5 openness and capability scores are deliberately not comparable across categories. For cross-category analysis, use the normalized `openness.class` → 3-bucket mapping.

## Comparison notes

Unlike [[Guardrails and Feedback Loops|Open Source Observer]], which provides the raw adoption metrics this project queries, the AI Stack Map adds an editorial judgment layer on top — it trades scale for trust.

Unlike automated benchmarks like [[FrontierCode]] or [[Sherlock Agent Eval]], which produce single-dimensional scores from test execution, the AI Stack Map produces multi-axis, human-judged scores measuring dimensions (openness, adoption quality) that automated systems can't capture.

Unlike Chip Huyen's Good AI List (a seed source for discovery), the AI Stack Map adds a rigorous scoring layer with primary-source citations and a gap analysis framework. It goes from "what exists" to "how good and how open is it, and what's missing."

The project's taxonomy descends from the 2024 Columbia Convening on Openness in AI and the Model Openness Framework (MOF), making it a practical implementation of academic openness taxonomies — similar in spirit to how [[The Agentic Product Standard v2.0]] operationalizes agent design principles into a measurable standard, or how [[Elements of Agentic Systems Design]] provides a ten-element taxonomy for agent systems.

The methodology's treatment of openness as an orthogonal axis (separate from capability/adoption maturity) parallels [[Load-Bearing Assumptions]]' approach of surfacing assumptions as independently verifiable claims rather than collapsing them into a composite score.

In terms of data pipeline architecture, it's a simpler but more rigorous version of the pattern seen in projects like [[Data Engineering for Large Models]] — a deterministic build pipeline over hand-curated source data, with validation as a first-class concern.

---
*Sources: [[raw/os-ai-map]]*
*Last updated: 2026-07-05*
