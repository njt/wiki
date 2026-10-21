---
url: https://github.com/joshuavial/codebase-inspector
title: "codebase-inspector (cbi)"
author: Joshua Vial
date_fetched: 2026-10-21
date_published: 2026 (v0.1.0)
topics:
  - developer-tools
  - agent-memory-and-context
---

# codebase-inspector (cbi)

`cbi` is a deterministic Python CLI that maps a git repository into a layered model — workspace, deployables, packages, folders, files, symbols, tests — stored in a SQLite database per repo (`.cbi/model.db`). It never calls a model and never executes the target's code. The judgement work (file summaries, concept names, structure confirmation) is handed to AI agents through a task protocol: `cbi tasks` lists open work, `cbi task <id>` prints a brief with a JSON schema, `cbi submit` validates and stores the answer. Humans browse the map in a static viewer or an Electron desktop shell.

Parsing uses tree-sitter (Python, TypeScript, TSX, JS, plus C#/Vue in tests); resolution builds `imports`/`calls`/`tests` edges with confidence scores; coverage (lcov, coverage.py JSON) and test results (JUnit XML) can be ingested to link tests to code by execution, not just naming. `cbi check` runs the whole pipeline model-free in CI and fails when changed files lack summaries or committed answers — team mode commits the "judgement half" of the map alongside the code and gates it like any other review. The architecture docs are unusually good: PRDs, ADRs, a decision log, and rejected alternatives (SCIP indexers need the repo's deps installed; a served viewer violates the `file://` constraint).

The interesting design bet is the division of labour: everything that can be derived deterministically is, and everything requiring understanding is packaged as small, cacheable, hash-stamped tasks that any agent can fill in — with answers cached per content hash in a shared cross-repo cache, and stale answers rejected by echoing `input_hash`.
