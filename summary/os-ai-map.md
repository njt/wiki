---
url: https://github.com/currentai-org/os-ai-map
title: Open Source AI Map
author: CurrentAI.org
date_fetched: 2026-07-05
date_published: 2025
topics:
  - ai-research-and-models
---

# Open Source AI Map — Architectural Analysis

## Project overview

The Open Source AI Map (`os-ai-map`) is the public data and modeling backend behind the [Open Source AI Stack Gap Map](https://map.currentai.org/), published by CurrentAI.org. It is a curated, version-controlled, hand-scored catalog of ~458 AI products across 15 categories, organized into a three-layer taxonomy descending from the [2024 Columbia Convening on Openness in AI](https://arxiv.org/abs/2405.15802). The repo holds the data, the deterministic build pipeline that validates and serializes it, and the marimo notebook that renders the published interactive map. There is no frontend in this repo; the website lives in a separate repository.

**Scale**: ~251 organization YAML files, ~474 product YAML files, ~474 score YAML files, 15 category YAML files, plus a single `taxonomy.yaml` manifest. The Python build pipeline is ~1,600 lines of code (`serialize.py` 290 LOC, `render.py` 1,122 LOC, `validate.py` 161 LOC, `slugs.py` 15 LOC). The pytest suite is ~400 lines across three files. The methodology documentation is 141 lines of hand-authored Markdown with computed placeholders.

**License**: MIT.

## Architecture pattern

The system follows a **curated data + deterministic pipeline** pattern, structured as four concerns:

### The four-concern data model

The curated source data lives in `sources/` as flat YAML files, organized into four concerns plus a manifest:

1. **Organizations** (`sources/organizations/<slug>.yaml`): Each org file declares `name` (slug), `display_name`, `type` (company/lab/foundation/individual/government/unknown), `homepage`, optional `github` typed-URL array, and a `products:` roster listing which product slugs belong to the org. The org *owns* the product roster — a product slug must appear in exactly one org file, validated at build time.

2. **Categories** (`sources/categories/<slug>.yaml`): Each category declares `name`, `display_name`, `description`, `strapline` (an editorial finding), `weights` (`adopt` and `cap` floats that sum to 1), an ordered `products:` roster, and optionally `disclosure_gap: true`. Category slugs use underscore form (`base_pretrained`, `finetuned_chat`, etc.).

3. **Products** (`sources/products/<slug>.yaml`): Each product file declares `name` (kebab-case slug), `display_name`, `type` (model/software/dataset/hardware), `description`, optional `comments` (version info, provenance notes), typed artifact arrays (`github`, `npm`, `pypi`, `crates`, `go`, `huggingface_model`, `huggingface_dataset`), and an optional `lineage` block (`derived_from`, `curated_with`, `trains`). Products do NOT carry an `org:` field — org membership is exclusively declared in the org file.

4. **Scores** (`sources/scores/<slug>.yaml`): One per product (same slug), with three axes: `openness` (`class`, `score` 0-5, `components`, `confidence`, `note`, `sources`), `adoption` (`level` 1-5, `reach`, `signal_type`, `confidence`, `note`, `sources`), and `capability` (`score` 1-5, `basis`, `value`, `confidence`, `note`, `sources`). Every non-null score value requires a `sources:` citation with `url`, `shows`, and `accessed` fields.

5. **Taxonomy manifest** (`sources/taxonomy.yaml`): Declares the three arcs (Columbia ontology layers: `product_ux`, `model_components`, `infrastructure`), each with `name`, `layer` slug, and an ordered `categories:` list. This single file owns the global display order — `serialize.py` derives both the display arc name and the machine layer slug from here.

### The validation constraint system

`build/validate.py` enforces a web of cross-file invariants that go well beyond JSON Schema validation:

- **Schema validation**: Each file type is validated against its JSON Schema (`docs/schemas/product.schema.json`, `score.schema.json`, `organization.schema.json`, `category.schema.json`, `taxonomy.schema.json`).
- **Roster completeness**: Every product slug must appear in exactly one category roster AND exactly one org roster. Orphaned products (in no roster) or duplicates (in two rosters) fail validation.
- **Taxonomy completeness**: Every category file must appear in exactly one taxonomy arc.
- **Openness class validity**: The `class` field must be valid for the product's `type` — e.g., `open_core` is only valid for software, `open_weights` only for models.
- **Stars cap**: Adoption with `signal_type: stars_fallback` cannot exceed level 3.
- **Source enforcement**: Non-null openness/adoption/capability scores must have at least one `sources:` entry.
- **Long-tail sync**: The frozen long-tail fixture's `counts.scored` must match the current product count, preventing the notebook from contradicting itself.
- **Category→score existence**: Every rostered product must have a corresponding score file, preventing runtime crashes in `serialize.py`.

Each product type (model, software, dataset, hardware) has its own openness class vocabulary, and each axis has its own set of valid `signal_type` and `confidence` values. The validation system rejects invalid combinations rather than silently accepting them.

### The serialization pipeline

`build/serialize.py` compiles the four-concern YAML sources into a single `build/notebook_data.json` payload consumed by the notebook renderer. The key computational steps:

1. **Product-org reverse map**: Walks every org's `products:` roster to build a `product_slug -> org_slug` map, since products don't carry org membership.

2. **Taxonomy-derived order**: Flattens `taxonomy.yaml`'s `arcs[].categories` lists into the global `order[]` array, and builds `{category_slug: arc_name}` and `{category_slug: layer_slug}` maps so both the display arc and the machine layer come from a single source.

3. **Product row enrichment**: Each product is enriched with its openness `bucket` (open/open-ish/closed, a 3-way collapse of the class vocabulary computed by `_gap_bucket()`), a weighted `maturity` score (adoption × w_adopt + capability × w_cap, normalized to 1-5, rounded to 2dp to eliminate float epsilon from stage decisions), and a canonical `mature` flag (fully open AND maturity ≥ 4.5).

4. **Category stage computation** (`_stage_and_gaps()`): The most architecturally significant function. It classifies each product by openness bucket, counts mature fully-open products, finds the best fully-open product's score, and assigns a stage 0-5 plus a set of gaps. The algorithm is **open-only**: only products in the `open` bucket count toward stage advancement; open-ish products only flag the openness gap. The exact cutoffs (4.5 maturity bar, 4 mature products for Stage 5, score bands at 3.5, 3.0, 2.0 for lower stages) are named constants at the top of the file.

5. **Long-tail filtering**: The frozen `_frozen_long_tail.json` sample is filtered to remove products that are now categorized, so the long-tail display never shows a product that's also in the scored set.

6. **Descriptions block**: Stage definitions, gap definitions, and per-category neutral descriptions are serialized into a top-level `descriptions` block so downstream consumers can render a legend without re-deriving the methodology.

The payload structure is: `{ descriptions, layer_order[], categories: {cid: {label, arc, layer, stage, gaps, products[]}}, order[], n_total, generated, long_tail }`.

### The renderer

`build/render.py` generates `notebooks/ai-stack-map.py`, a ~1,100-line marimo notebook. The approach is template-based code generation rather than dynamic rendering:

1. **Methodology injection**: Reads `docs/methodology.md` (hand-authored, canonical), substitutes `{placeholders}` with live counts computed from the payload (e.g., `{total}`, `{scored}`, `{n_orgs}`, `{n_citations}`), converts Markdown to HTML via Python's `markdown` library, and bakes the HTML into the notebook template.

2. **Source-derived literals**: Reads straplines and weights from `sources/categories/<cid>.yaml` at render time and generates Python dict literals, eliminating hardcoded data in the notebook template.

3. **Section cell generation**: Dynamically generates one `@app.cell` function per category (15 cells total), each calling a shared `render_section()` helper that builds the full HTML table with openness bars, adoption/capability bar charts, verdict spine, and scoring callout.

4. **JS-driven interactivity without a kernel**: The rendered notebook's filter/sort controls and details modals are implemented entirely in JavaScript, injected via hidden iframe `onload` handlers (marimo strips `<script>` tags in static exports). This means the interactive map works in static HTML exports with no running kernel.

5. **Design system**: A hand-tuned typography system (Noto Serif for headlines, Plus Jakarta Sans for body, DM Mono for data), a salmon-ramp color palette for openness that forms a single ordered scale, and sharp corners throughout (deliberately avoiding rounded corners).

### Adjacent systems

The repo also contains:
- **Warehouse** (`warehouse/`): UDM SQL models and Python fetchers for adoption/activity signals, powered by `pyoso` (Open Source Observer client). Contributors work read-only here; only maintainers write.
- **Skills** (`skills/`): Four Claude Code editor skills that mirror contribution recipes: `curate-category`, `add-product`, `add-data-source`, `pyoso-analyst`.
- **Standalone notebooks**: `pypi-geo-trends.py`, `oss-ai-trends.py`, and `long-tail-explorer.py` query the `currentai.*` warehouse tables live via `pyoso` and are not part of the build pipeline.

### CI/CD

GitHub Actions workflows: `validate.yml` runs `build.validate` + `pytest` on every PR; `regenerate.yml` regenerates generated files on merge to main; `refresh-data.yml` runs warehouse fetchers on a schedule.

## Key techniques and non-obvious implementation choices

### 1. Openness as an orthogonal axis, not a linear spectrum

The single most important design decision: openness is NOT a point on a continuous maturity scale. It is an **orthogonal axis**. A category can have strong, widely-adopted products that aren't fully open (e.g., open-weights models like Llama), and the system explicitly reports this as an "openness gap" rather than downgrading the category's maturity. This is the distinction the map exists to surface.

The implementation uses a three-bucket collapse (`open` / `open-ish` / `closed`) derived from the fine-grained `openness.class` vocabulary. Only the `open` bucket counts toward stage advancement. Open-ish products (open_weights, source_available, gated, open_toolchain) are used solely to detect the openness gap. The consequence is bounded but real: counting open-weights as fully open would move only 3 of 15 categories, all in the model layer.

### 2. The disclosure gap — a declared attribute, not inferred

The `disclosure` gap is set with `disclosure_gap: true` in the category YAML file. It is NOT derived from product scores. This is deliberate: it describes the *closed* frontier's silence (labs publishing neither proprietary data nor their exact recipe), which the open products' scores cannot express. Making it a declared attribute prevents it from silently toggling when the roster changes. It can appear at any stage, including Stage 5 (mature open ecosystem), because it describes the competitor's opacity, not the open ecosystem's weakness.

### 3. Float-epsilon-aware maturity scoring

The maturity score computation uses `round(value, 2)` before comparison against threshold boundaries. This prevents float noise (e.g., `0.3 * 3 + 0.7 * 3 = 2.9999999999999996`) from pushing a product across a stage boundary. The test suite explicitly validates this: `test_stage_uses_rounded_maturity_not_float_noise`. The stage thresholds compare the same 2dp value that's displayed.

### 4. Source-citation-as-requirement, not scoring-as-formula

Unlike a naive approach that would compute scores from downloaded metrics, every score in this system is an editorial judgment made by a human analyst. The pipeline enforces that every non-null score has at least one primary source citation (with URL, what-it-shows, and access date), but it does NOT compute scores. The methodology is explicit about this: "The scores themselves are editorial judgments rather than the output of a deterministic formula, so this is an open trail to follow, not an independently reproducible computation." This makes the map auditable but not mechanically reproducible.

### 5. Code generation as rendering, not dynamic templating

The renderer (`build/render.py`) generates a Python file (the marimo notebook) from the JSON payload. It doesn't use a template engine with conditional logic — it builds Python code as strings, with the methodology HTML baked in, the section cells dynamically constructed, and the JS interactivity inlined. This is a deliberate choice: the generated notebook is self-contained (data embedded, no API calls), which makes it static-export-friendly. The methodology numbers are substituted at render time from live payload counts, so the prose never drifts.

### 6. The long-tail self-healing filter

The frozen long-tail fixture (`build/_frozen_long_tail.json`) contains a `top` sample of the most-used uncategorized artifacts. When `serialize.py` runs, it filters out any samples whose artifact IDs now belong to categorized products (by matching GitHub repo names, PyPI/npm package names, HuggingFace model/dataset IDs). This means the long-tail display self-heals: as products are scored and added to categories, they automatically disappear from the uncategorized list without manual cleanup of the fixture.

### 7. Product→org membership normalization

The project made a deliberate architectural choice: products do NOT carry an `org:` field. Instead, the organization file owns the `products:` roster. This preserves a single source of truth for membership: a product can't accidentally claim a different org than the org claims for it. The `serialize.py` pipeline reverses the map at build time, walking every org roster to build `product_slug -> org_slug`. Validation enforces that every product slug appears in exactly one org roster.

### 8. Explicit policy parameters as named constants

The stage and gap computation (`_stage_and_gaps`) uses named constants at the top of `serialize.py` for every tunable threshold: `_MATURE_MIN = 4.5`, `_STAGE5_MIN_MATURE = 4`, `_CAPABLE_MIN = 4`. These are described as "deliberate, tunable choices rather than fixed law" and "should be reviewed when the scoring rubric or the curation density changes materially." This treats the scoring model as a tunable instrument rather than a fixed formula.

### 9. The openness class vocabulary is type-dependent

Different product types use different openness vocabularies:
- **Model**: `open_source`, `open_weights`, `restricted`, `closed`
- **Software**: `open_source`, `source_available`, `open_core`, `closed`
- **Dataset**: `open`, `gated`, `documented_only`, `closed`
- **Hardware**: `open_hardware`, `open_toolchain`, `documented`, `restricted`

The `validate.py` function `OPENNESS_CLASSES` enforces this at build time: a score with `class: open_core` on a `type: model` product fails validation. The renderer later collapses all vocabularies onto a single 12-entry gradient and a 3-bucket verdict.

## Design trade-offs

### Optimized for: auditability over automation

The project is fundamentally optimized for auditability. Every score cites a primary source with an access date. Scores are editorial judgments, not computed outputs. This makes the map defensible and traceable, but it means adding a product requires human research time, not just API calls. The trade-off is deliberate: the map's value proposition is that it's *trustworthy* rather than *comprehensive*.

### Optimized for: openness purity over inclusiveness

Only fully-open products advance a category's maturity stage. Open-weights models, no matter how capable or widely adopted, cannot move the needle. This makes the map a deliberately strict reading of the open ecosystem. The methodology acknowledges this is "consequential but bounded" — only three categories would change stage if the rule were relaxed. The trade-off is editorial clarity: the map takes a position rather than being neutral.

### Optimized for: static export over dynamic queries

The rendered notebook embeds all data and uses JS for interactivity. There is no API backend, no database queries in the published map. This makes it fast, offline-capable, and trivially deployable as static HTML. The cost is that the map is a snapshot, not a live dashboard — data refreshes require a full rebuild and redeploy.

### Optimized for: single source of truth over convenience

The taxonomy manifest (`taxonomy.yaml`) is the single source for display order, arc grouping, and layer assignment. Category files don't have `arc` or `order` fields. The methodology Markdown is the single source for prose, with numbers substituted at render time. Straplines and weights live in category YAML, not the notebook template. This means changing the display order requires editing only one file, but it also means render-time failures if placeholders aren't substituted.

### Sacrificed: cross-category comparability of raw scores

The openness and capability 0-5 scores are deliberately NOT comparable across categories. A pretrained model at openness 2 is genuinely restricted, while a deployment tool at 2 is something else entirely. The guide explicitly warns: "Don't compare the numbers directly across categories." For cross-category analysis, you use the normalized `class` → 3-bucket mapping or the 12-entry class-to-gradient table.

## Core abstractions

1. **The four-concern data model**: Organizations, Categories, Products, Scores — each with its own YAML directory, JSON Schema, and cross-file invariants. Products are the central entity; everything else references them by slug. The separation of concerns (product metadata ≠ scoring ≠ membership) prevents any single file from being a god object.

2. **The openness bucket**: The 3-way collapse (`open` / `open-ish` / `closed`) is the system's central abstraction. It bridges the fine-grained 12+ class vocabulary (which is type-dependent) with the stage computation (which is type-independent). Everything downstream — mature flag, stage counting, gap detection — operates on this bucket.

3. **The maturity score**: A per-product weighted blend of adoption and capability, normalized to 1-5, that serves as the single input to the category stage ladder. The weights vary per category (adoption-heavy for end-user surfaces, capability-heavy for model categories), making it a per-category composite rather than a universal metric.

## Comparison to related projects

Unlike **Open Source Observer** (OSO), which this project uses as a data source for warehouse queries, the AI Stack Map is a curated editorial product rather than a data aggregation platform. OSO provides the raw metrics; the AI Stack Map provides the human judgment on top.

Unlike **Chip Huyen's Good AI List**, which is an open catalog seeded from GitHub topics, the AI Stack Map adds a rigorous scoring layer with source citations and a gap analysis framework. It goes from "what exists" to "how good and how open is it, and what's missing."

Unlike automated benchmarks like **SWE-bench** or **Chatbot Arena**, which produce single-dimensional scores from test execution, the AI Stack Map produces multi-axis, human-judged scores that capture dimensions (openness, adoption quality) that automated systems can't measure.

The project's methodology descends directly from the [2024 Columbia Convening on Openness in AI](https://arxiv.org/abs/2405.15802) and the [Model Openness Framework](https://arxiv.org/abs/2403.13784) (MOF), making it a practical implementation of academic openness taxonomies rather than an ad-hoc classification.

## Architecture weaknesses

1. **Scoring bottleneck**: Editorial scoring doesn't scale. At ~458 scored products out of an ~10K+ discovered universe, coverage is ~4.5%. The long tail grows faster than human analysts can score it.

2. **Editor stake problem**: The methodology calls for editors "with no direct stake in the products they rank," but the map is maintained by CurrentAI.org — an organization presumably with its own perspective on open AI. The roadmap acknowledges this as unresolved.

3. **Snapshot latency**: The map is a static build. Product scores drift; usage changes. Between rebuilds, the published map can be outdated. The methodology doesn't state a refresh cadence.

4. **Single-dimensional capability**: Capability is a single 1-5 score, but "capability" for a model (benchmark performance) is fundamentally different from "capability" for a dataset (training value from downstream evidence) or for deployment software (feature coverage). The same score means fundamentally different things.

5. **English-centric product descriptions**: The methodology acknowledges this limitation. Product discovery is biased toward English-language artifacts and English-centric registries.
