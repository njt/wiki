# Agent-Building Playbook (Ramparte)

An open-source, git-native pattern language for building agentic software: 113 curated patterns in one canonical markdown source, rendered two ways for two audiences (a dense agent-facing index and a human-browsable wiki), plus a radial "flavor wheel" taxonomy that projects the whole field onto 6 families → 13 dimensions → 113 patterns. It matters because it is one of the few *machine-ingestible* pattern catalogs in the agent space — the format, not any single pattern, is the contribution.

---

## Architecture

The whole repo is a **single-source, multi-view pipeline** — no runtime, no code beyond build scripts. The source of truth is `patterns/` (113 files), each a flat-frontmatter markdown "pearl." Everything else is derived:

```
                       scripts/build-index.sh  ->  INDEX.md         (agents, dense)
patterns/*.md  ------<
                       .wiki/scripts/publish.sh ->  wiki/            (humans, HTML)
                       scripts/build-wheel.py    ->  wheel/           (radial taxonomy)
```

Three build artifacts, three audiences, zero hand-edited output. The `.wiki/context/schema.md` makes this explicit — it is a *render/publish* layer, "not an ingest-and-synthesize loop."

- **`scripts/build-index.sh`** (bash + awk) — extracts the three frontmatter fields per file, emits one tab-delimited row per tag, groups by `sort -u` tag then sorts each tag's patterns by title, and writes atomically (`mktemp` + `mv`). Two non-obvious details: it pins `LC_ALL=C` so macOS/CI locales don't churn the index, and it **fails loud**, naming the exact file missing a required field.
- **`scripts/build-wheel.py`** (stdlib-only Python) — the most interesting artifact. A curated `FAMILIES` dict maps 6 families to 13 dimensions; each pattern gets **one primary home** (its first-listed dimension, overridable via `PRIMARY_OVERRIDES`), and every other tag it carries is preserved as an `also:` cross-reference. It hand-rolls a YAML emitter (no PyYAML), shades the wheel with `colorsys` HSL, and emits `taxonomy.yaml`, `wheel.md`, a Graphviz `wheel.dot`, `wheel.svg`, and an interactive `explore.html`.
- **`.wiki/scripts/publish.sh`** — a heredoc-embedded Python program renders `patterns/*.md` into `wiki/index.html` + `wiki/patterns/{kebab}.html` with an inline CSS design system, packaging a zip under `.wiki/dist/`.

The **schema** is the true core abstraction. One entity — `pattern` — identified by its filename (`{kebab-title}.md`), no `id` field. Frontmatter is *flat*: exactly three scalar lines (`title`, `one_liner`, `dimensions`), where `dimensions` is a multi-value comma-separated tag. The body follows a conventional pearl: **What it is → When to reach for it → When NOT to → Exemplars → Related**.

## Key techniques

- **The filename is the identifier.** No `id` field, no database — the kebab slug is the join key across `INDEX.md`, `wiki/patterns/{slug}.html`, and the wheel's `taxonomy.yaml`. This is what makes cross-linking trivial and hand-edits impossible to get out of sync.
- **Flat frontmatter as a machine contract.** Forcing exactly three inline scalar lines (no nested YAML) is a deliberate constraint: it makes the corpus parseable by a 15-line awk extractor *and* by agents, and it prevents authors from smuggling structure that breaks the generators. The cost is that richer metadata (exemplars, related links) is demoted to the body, where it's conventional but unvalidated.
- **"One primary home, `also:` everywhere else."** The wheel solves the many-tags problem by giving each pattern a single canonical dimension (first-listed) and rendering secondary tags as cross-references — so the taxonomy is a tree, not a many-to-many graph, while still preserving the "appears under every tag" faceting in `INDEX.md`.
- **The `When NOT to` contraindication.** Nearly every pattern carries an explicit "when not to use this" — the field Alexander's originals lacked, and the same differentiator this wiki tracks elsewhere. It's what turns a list of advice into something debuggable: a pattern with a sharp contraindication is a decision rule, not a slogan.
- **Deterministic rails around the corpus itself.** The build scripts *practise* what the patterns preach: fail-loud validation (`missing frontmatter field` → exit 1), atomic writes, deterministic collation, and generated-files-you-must-not-edit enforced by a GitHub Action on merge. A "curated collection" that couldn't regenerate its own index would be exactly the kind of stale-artifact hazard the corpus warns against.

## Design decisions

- **Curation over scale.** 113 patterns is tiny next to [[Software Engineering Practice Atlas]]'s 4,654 cards or [[Agentic Design (Pattern Catalog)]]'s 280+. The trade is deliberate: every pattern is human-authored, carries usage guidance *and* a contraindication, and passes a PR review. This is the "human-curated depth" end of the curation-vs-generation axis.
- **Open and git-native over proprietary.** Unlike KORTEXYA's searchable web catalog with a freemium Pro tier, this is plain markdown in a public repo: PR-based contribution, versioned diff history, forkable, and free to point an agent at. The downside is no interactive search/demos — discovery is limited to `INDEX.md` skimming and the wheel.
- **Flat taxonomy vocabulary that grows by review.** Seed vocabulary was 10 tags; it's now 13 (`human-factors`, `intent`, `knowledge` added), with the rule "propose a new tag in the PR, never invent silently." This keeps the faceting axis disciplined instead of exploding into a hundred bespoke tags.
- **No provenance weighting.** External literature, session history, and the Amplifier team's own experience are treated as peers — a v1 simplicity call that sidesteps the citation-credibility problem at the cost of not telling you *why* to trust a given pearl (the exemplars list is the only provenance signal).
- **The dual-audience bet.** Making `INDEX.md` dense and token-efficient for agents while rendering `wiki/` for humans is a bet that *agents will read this repo directly* — i.e. that the corpus is itself a tool, not just documentation about tools.

## Comparison notes

- **vs [[Agentic Design (Pattern Catalog)]]** — same Alexander-style move, opposite execution: KORTEXYA is a proprietary, searchable web catalog (280+ patterns + freemium Prompt Optimizer/Eval Lab), while this playbook is an open git-native markdown corpus with a validated flat-frontmatter contract. KORTEXYA optimizes for *browsability*; Ramparte optimizes for *machine-ingestibility and contribution*.
- **vs [[A Pattern Language (Christopher Alexander)]]** — the direct ancestor. The playbook keeps the "name the recurring problem + proven solution" grammar but adds the **contraindication** field Alexander lacked, and — the genuinely new bit — renders the language in *two* forms so a non-human reader (an agent skimming `INDEX.md`) consumes the same source a human does.
- **vs [[Agent Orchestration]]** — the playbook's Orchestration family (16 patterns: workflows-vs-agents, single-threaded-default, three-primitives, match-topology-to-the-work) is a *named-pattern* encoding of findings this wiki's orchestration hub reaches independently (hierarchy over flat coordination, planner/worker/judge). The playbook generalizes those into transferable rules with explicit "when NOT to" escape hatches.
- **vs [[Guardrails and Feedback Loops]]** — its Trust family (verification/reliability/observability) is the same "deterministic beats probabilistic" thesis: critic applications, deterministic rails, fail-loud harnesses, proposer/authority separation, auditable artifacts. The playbook phrases them as composable pearls rather than as a single enforcement hierarchy.

---

*Sources: [[raw/agent-building-playbook]], [[summary/agent-building-playbook]]*
*Last updated: 2026-08-22*
