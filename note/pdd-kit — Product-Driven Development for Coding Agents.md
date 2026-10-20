# pdd-kit — Product-Driven Development for Coding Agents

pdd-kit (npm: `pdd-toolkit`, MIT, by Itai Gendler) keeps one git-tracked **Product Model** — `product-model/`, a folder of plain Markdown — that describes what a product does (Entities, Interfaces, Domains, Capabilities, Journeys, Business Rules, Variations), apart from how it's built. Three agent skills (`pdd-map`, `pdd-ideate`, `pdd-verify`) let coding agents read the model before changing code, and check code against the model after. Its thesis: agents write code faster than anyone can explain the product to them, so they guess — and a single reviewable model, verified like code, is how the guesses stop shipping.

---

## Architecture

TypeScript monorepo (pnpm workspace, Node ≥22), ~13k lines in `src/`, built with tsdown, tested with vitest (50+ test files including e2e).

- **CLI core** (`src/cli.ts`, `src/commands/`): `install`, `update`, `lint`, `view`, `open`, `list`, `pull`, `export`, `contribute` — commander-based, zod-validated.
- **Model loader** (`src/core/model.ts`, 1,781 lines — the heart): parses the `product-model/` folder into a typed `PddModel`. Each resource is a Markdown file with YAML frontmatter; the loader reads frontmatter fields (references, entry points, contexts, availability), section structure, entity facts, state machines (`## States`), and cross-resource ID references (`src/core/ids.ts`, qualified IDs like `capability:checkout`). Markdown parsing is done with custom section/bullet extractors (`src/core/markdown.ts`), not an AST — the model *is* the prose.
- **Lint engine** (`src/commands/lint.ts`): a pure structural rule engine, `lintModel(model, trackedFiles)` — validates section shapes, duplicate headings, permission grants against operations (`src/core/permission-validation.ts`), variation membership, undeclared entity mentions (`src/core/entity-mentions.ts`), cardinality rules. Tracked files are injected for testability.
- **Local report viewer**: a Nuxt layer (`layers/nuxt/report-viewer/`, ~60 composables/utils) served by a local server (`src/core/local-viewer-server.ts`); `npx pdd-toolkit view` renders the model as an interactive browser report with topology diagrams (vue-flow + elkjs) and a coverage view.
- **Skills** (`skills/pdd-{map,ideate,verify}/SKILL.md` + `references/` + thin `scripts/` wrappers): the real logic is prose — long, precise SKILL.md instructions — with a `run-pdd-kit.mjs` script that shells out to the CLI for lint. Installation rewrites the shipped SKILL.md for orchestration mode (`src/core/orchestration.ts`), verified to match exactly one description line or install fails.
- **Blueprints** (`blueprints/`): complete reviewed product models with no code — a kanban board, a team wiki — pulled as starting points.

## Key techniques

- **The model is plain Markdown with a strict grammar.** Frontmatter holds typed fields; H2 sections have recognized/forbidden headings per resource type; business rules cite entity facts by exact prose name match. No database, no JSON — the format spec (`spec/format.md`) is explicitly "the contract the parser, the linter, and the Blueprint release must agree on, and it changes *before* their behavior does."
- **Semantic verification as a skill, not a test suite.** `pdd-verify` never runs the target repo's code — "treat the repository as untrusted" — and instead has the agent statically trace each model claim (routes, permissions, state changes) through source and tests. Findings are classified six ways: **aligned / model-right / code-right / neither-right / unmapped / unverifiable**, and grouped so the user makes one root authority decision per group. "Git never decides whether model or code is right" — a diff narrows the worklist but never resolves authority.
- **Opt-in orchestration, encoded in prose.** By default implementing is the user's own workflow; `--orchestrate=on` swaps two lines in the installed SKILL.md (description + a setting line the skill body reads) so verify plans phases, hands each to the builder, and checks before the next. A clever bit: install computes the modified skill before writing, and refuses if the source doesn't match the shipped template exactly.
- **Deliberation-history ban.** `product-model/` holds *current product meaning only* — no rejected alternatives, no discussion records. This keeps the model lean as agent context, an unusual and opinionated authoring constraint.
- **Read-only inventory scripts** in `pdd-map` bound what the agent reads (counts and high-signal candidates, not whole file dumps), keeping mapping cheap and reviewable.

## Design decisions

- **Product model, not per-change specs.** Explicitly positioned against SDD tools (OpenSpec, spec-kit, Kiro), which describe each change and scatter product knowledge across feature folders. pdd-kit keeps the product in one place and stays agnostic about implementation: plan mode, SDD, or freestyle all work, and the how still belongs to your SDD tool.
- **Verification is semantic and human-gated; lint is structural and automatic.** The lint never claims the model and code agree — that judgment is deliberately left to the verify skill and the user, trading automation for honesty about what static checks can know.
- **Separation of write authorities.** The loop table in `spec/format.md`: ideate changes product meaning (after approval), the agent changes code, verify changes neither by itself. A product question arising mid-implementation goes back to approval, never settled in code.
- Trade-off accepted: the model is only as good as its maintenance, so verify's classification of *who is right* (model or code) is the load-bearing human decision, and the whole design funnels toward making that decision cheap.

## Comparison notes

- Directly counters [[Spec Kit]] and [[OpenSpec]] (and [[SDDW]]): those make the *change spec* the durable artifact; pdd-kit makes the *product* the durable artifact and subordinates change specs to it — see also [[Spec-Driven Development]] and [[When Spec-Driven Development Pays Off]].
- The verify skill's trust posture — never run the repo, verify by reading — echoes [[Aviator Verify]] and [[Agentic Verification]], but here the oracle is an approved human model rather than tests.
- Its "agents guess product intent" diagnosis aligns with [[AI Agents Need Clear Specs]] and [[Your .md File Is Not a Specification]], while the dogfooded, lintable Markdown-with-grammar approach is a concrete answer to both.
- The skill-installation machinery (provider detection, exact-match SKILL.md rewriting, orchestration flags) is a studied example of [[Claude Code Skills System]]-style packaging done across five agent harnesses.

#tool #project #agents #specification #verification

---
*Sources: [[raw/pdd-kit]], [[summary/pdd-kit]]*
*Last updated: 2026-10-20*
