# Cassis Ontology Skills — Evidence-Grounded Warehouse Ontology Bootstrap

GetCassis/ontology-skills ships two Claude Code Agent Skills and the Python "bootstrap kit" behind them: a pipeline that assembles the first version of an analytics agent's context — an ontology of domains, tables, columns, metrics, and joins — from the schema exports, dbt projects, dashboards, query logs, and docs a data team already owns. Its governing principle is "a blank beats a guess": nothing enters the ontology without evidence attached, and everything the run cannot settle ships as a stated assumption rather than an invented fact.

---

## Architecture

A three-command driver (`bootstrap.py` prep/build/finish) sequences ~16 single-purpose scripts (about 7,700 lines of pure Python; dependencies are just PyYAML, sqlglot, ruamel.yaml). The design rule, stated in the code and enforced throughout: **if a step's inputs fully determine its output, it is a script, never an agent.**

- `ingest.py` normalizes inputs: schema CSVs/JSON (case-blind headers, comment columns, declared keys), a dbt adapter with `{{ doc() }}` block resolution (`dispatch_prep.py resolve-docs`), dbt-docs exports, dashboard exports (Metabase and a cards-YAML shape), query history, and free-form docs/context. An LLM config fallback handles unrecognized repo layouts.
- `evidence.py` mines everything deterministically: enums, grains, join candidates from SQL text (regex-based alias-aware FROM/JOIN mining plus dbt declared/implicit relationships), metric candidates (aggregates occurring ≥3 times), usage rankings, and "non-summable" candidates (GROUPING SETS, stacked UNION ALL blocks). Everything is a *candidate with provenance IDs*, never an assertion.
- `scope.py` does two-level scoping: level 1 is a classifier that discards the non-business surface (staging mirrors, seeds, export schemas — 59% of one real warehouse's columns) while keeping it as *evidence*; level 2 is the human's cut.
- `dispatch_prep.py` emits one markdown packet per enrichment batch — schema, grain, enums, tests, resolved descriptions, defining SQL — so each subagent reads exactly one file instead of globbing the repo.
- Agent stages (B1 domain tree alone on Opus; B2 joins/metrics in parallel Sonnet; C one column-enrichment agent per packet) interleave with the checkpoints; `verify.py` gates the result; `emit.py` writes the canonical tree and the dbt merge.

## Key techniques

- **Deterministic verification gates over generated prose.** `verify.py` runs eight gates: identifier resolution (with a clever structural rule — any token before an open paren is a function, not a column, ending a hand-maintained function-list failure generator), restatement detection, hedge-word scanning, prose references to nonexistent tables, metric grounding against the mined corpus, glossary cross-entity contamination, divergent duplicates (the same owner sentence corrected on one table but left verbatim on another — a bug class born of per-packet agent isolation), and "population rules" stranded away from the tables they govern (found via selective boolean column terms + BFS over the join graph).
- **Judge input extraction.** `verify.py` pairs every claim containing risk keywords ("computed as", "derived from", "filter on") with the defining SQL snippet, for an LLM judge pass. This judge is the only stage that has caught a description contradicting its own SQL — on two different warehouses.
- **Provenance tiers and corroboration stamps.** Every description records where it came from (glossary tier, warehouse comment, agent-drafted); every metric gets UNGROUNDED / VERIFY / corroborated-by-2 based on dashboard and query-log corroboration. A first run without those sources stamps everything VERIFY by design — and business synonyms ("DAU") wait until a human confirms them.
- **Structured unknowns with a publish gate.** `questions.py` formalizes open questions into six tiers (blocker, ruling, provenance, finding, unbound, standard); blockers name the metrics they poison, answering is a file edit plus a free re-run, and zero question files is a *failure*, never a clean pass — a measured silent-failure mode (a green gate while the authored prose referenced a QUESTIONS.md that did not exist).
- **Silent-loss paranoia.** The code is dense with measured postconditions born of real failures: doc IDs keyed by basename collapsed 1,708 of GitLab's 4,736 handbook pages; bare-vs-qualified name mismatches left 0 of 114 grains matching; 3,399 rows with blank data_type; empty evidence classes are treated as bugs until proven otherwise. `evidence.py` fails loud when the repo/schema join is empty rather than letting a later phase propose "a scope of nothing."
- **Idempotent dbt merge.** `emit.py --emit dbt` uses ruamel.yaml round-trip loading so comments and key order survive, writes only where the project has no description, and namespaces everything else under `meta.cassis.*`. Measured limits: metrics export only with a MetricFlow time spine, and mandatory-filter metrics never export as dbt metrics because the aggregate without its filter is "a wrong number that looks governed."

## Design decisions

The kit optimizes for trust and auditability over coverage, and for human attention as the scarcest resource. Reviews are batch artifacts (files read top to bottom), never per-item interrogation in chat; cost is printed before each phase, measured from real session transcripts rather than projected; most of a run's list price is cache re-reads, which they say plainly. The trade-off accepted: join mining is regex-based (undercounts CTE-heavy SQL), the restatement check is lenient, corroboration over-flags, and there is deliberately no evaluation harness — "the kit cannot tell you whether the ontology it produced is good; the four checkpoints and the reports are how you tell." It also refuses to harvest definitions from prose: applying handbook prose to columns by name filled 8 columns and got 8 wrong, so a column dictionary must be declared by a person with `--glossary-dir`.

## Comparison notes

- Where [[Text-to-SQL in the Real World]] diagnoses enterprise warehouses humbling LLMs to ~10%, this kit is the constructive complement: it manufactures the missing context (grain, joins, defined metrics, vocabulary) that makes text-to-SQL tractable, using the agent to *read* the warehouse rather than query it — the same machine-reads/human-decides split as [[AI-Assisted Database Work — The Machine Reads, The Human Decides]].
- [[Vedana — Domain Models as Agent Context]] argues agents need a hand-built domain model; Cassis agrees on the destination but mechanizes the bootstrap, generating a reviewed first draft of that model from evidence rather than expert interviews.
- The verification architecture rhymes with [[Bondsmith — Hard Bonds for Soft Claims]]: no LLM claim counts without a deterministic proof, with the judge reserved for exactly the checks scripts cannot do. It differs in that Cassis ships the soft claims *with* their assumptions rather than blocking the whole run.
- Against [[Databricks Genie Spaces for SQL Analysts]], which rebuilds the semantic layer as prompt context inside one vendor's platform, Cassis emits plain reviewed files in the team's own repo and vocabulary — a semantic layer as code, with the defect list as a byproduct.

Tags: #tool #project #agents #database #data-quality #guardrails

---
*Sources: [[raw/ontology-skills]], [[summary/ontology-skills]]*
*Last updated: 2026-11-03*
