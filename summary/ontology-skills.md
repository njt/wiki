---
url: https://github.com/GetCassis/ontology-skills
title: "Cassis ontology skills"
author: GetCassis
date_fetched: 2026-11-03
date_published: 2026 (recent; expansion skill newly published)
topics:
  - databases-and-data
  - guardrails-and-feedback-loops
---

Cassis's ontology-skills repo ships two Claude Code Agent Skills plus a ~7,700-line pure-Python "bootstrap kit" that assembles the first version of an analytics agent's context — an ontology of domains, tables, columns, metrics and joins — from material a data team already has: warehouse schema exports, dbt projects, dashboards, query logs and docs. The method was measured end-to-end on GitLab's public analytics project (2,921 dbt models).

The kit's central discipline is that scripts do everything the inputs determine, and agents only do what needs judgment. A deterministic pipeline (ingest → evidence mining → scoping → skeleton → prefill → packets) feeds agent enrichment stages, and a human decides at exactly four checkpoints: scope, domain tree, metrics, and the blocking questions. Every claim ships with provenance; model-written prose is never treated as evidence — harvesting column definitions from a 4,700-page handbook filled 8 columns and got all 8 wrong.

Output is a reviewable file tree (YAML per table, per metric, domain READMEs) plus evidence reports: metric corroboration stamps (UNGROUNDED/VERIFY/corroborated), prefill reports showing how much was already the team's own words, and a QUESTIONS.md that doubles as a defect list for the customer's own pipeline. An optional `--emit dbt` merges descriptions back into the dbt project's own schema.yml additively and idempotently (via ruamel.yaml round-trip parsing), with measured limits around MetricFlow time spines and mandatory-filter metrics.

Costs are stated, not projected: a ~10-table sample run costs roughly 0.6M tokens / ~$30–60; a full 20–30-table increment ~1.6M tokens across an Opus driver plus ~20 Sonnet subagents, most of it cache re-reads. Verification is eight deterministic gates (identifiers, restatement, hedges, prose refs, metric grounding, glossary crossmatch, divergent duplicates, population rules) plus an LLM judge pass over claim/SQL pairs — the only stage that has caught descriptions contradicting their own SQL.
