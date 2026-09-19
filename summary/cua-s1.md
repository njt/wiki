---
url: https://github.com/trycua/cua/tree/main/libs/cua-s1
title: "Cua-S1"
author: trycua / Cua project
date_fetched: 2026-09-19
topics:
  - coding-agents-and-frameworks
  - security-and-sandboxing
---

Cua-S1 is a research project inside the trycua/cua monorepo (`libs/cua-s1`) for studying small, specialist computer-use models — deliberately scoped to a defined class of interface tasks rather than a generally capable computer-use agent. The first checkpoint in the family is `cua-s1-form-v0`, a specialist for form-oriented UI tasks. This is a source-only release: Python model, synthetic-data, training, evaluation, and optional Cua Driver integration code, with no model weights, datasets, demo binaries, or recordings published, and no checkpoint performance claim established. Source is MIT; future weights would carry separate artifact-specific terms.

The runtime's design is the substance of the document. Planning and execution are separate; the optional runtime defaults to a dry run, requires one unambiguous target window, uses snapshot-bound element tokens, and reobserves the window after each mutation. `execute` and `submit` are independent opt-ins, and submission is deliberately narrow: `submit=true` permits at most one high-confidence `Button` or `AXButton` whose normalized label is exactly `Submit` or `Submit Form`, with all other click decisions omitted. PDF access is confined to configured allowed roots (defaulting to the current working directory, with a recommendation for a dedicated least-privilege directory in production). Checkpoints load only from a local `safetensors` file plus matching JSON configuration; pickle-based PyTorch checkpoints are rejected.

The optional `cua-s1-mcp` server (stdio transport) is described as an advanced integration surface, not a configured model service. `CUA_S1_PLANNER_FACTORY` points at trusted Python code in `module:attribute` form — importing it executes code with the server process's privileges, so it must never target untrusted modules. `CUA_S1_ALLOWED_PDF_ROOTS` bounds what the server may read. The connected Cua Driver must provide exact-window snapshots, snapshot-bound element tokens, and confirmed action effects; because the portable driver contract does not expose `set_value`, fill execution fails closed unless a runtime explicitly advertises compatible token-based value mutation. MCP tool results and stdio logs are to be treated as sensitive, since they can contain values extracted from PDFs, form labels, and the values selected for entry.

Evaluation is abstention-aware: the included offline metrics distinguish accuracy, abstention, coverage, wrong actions, wrong targets, and actions taken when the expected behavior was to abstain, with synthetic train/validation/test splits separated by form signature. A future checkpoint release must add an untouched holdout, artifact hashes, exact environment details, and independently reproducible results. The deployment guidance is explicit: run computer-use models in isolated environments with least-privilege credentials and explicit action boundaries, verify important outcomes independently, and require human review before consequential, irreversible, financial, legal, medical, account, permission, or external-communication actions.
