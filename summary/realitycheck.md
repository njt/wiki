---
url: https://github.com/lhl/realitycheck
title: "Reality Check — A framework for rigorous, systematic analysis of claims, sources, predictions, and argument chains"
author: Leonard Lin
date_fetched: 2026-05-14
date_published: 2026
topics:
  - agent-memory-and-context
---

# Reality Check

A framework for rigorous, systematic analysis of claims, sources, predictions, and argument chains. Unified knowledge base with structured epistemic tooling.

v0.3.4, 458 tests. Python 91.4%, Jinja 5.3%, Shell 2.4%. Apache 2.0 license.

## Overview

"With so many hot takes, plausible theories, misinformation, and AI-generated content, sometimes, you need a realitycheck."

Reality Check enables users to build a unified knowledge base with:
- Claim registry tracking evidence levels and credence scores
- Source analysis using a 3-stage methodology (descriptive → evaluative → dialectical)
- Evidence linking connecting claims to sources with strength ratings
- Reasoning trails documenting epistemic provenance
- Prediction tracking with falsification criteria
- Argument chain mapping for logical dependencies
- Semantic search across the knowledge base

## Technical Architecture

- Embedding model: `all-MiniLM-L6-v2` (384-dimensional vectors)
- Storage: LanceDB
- GPU support: NVIDIA CUDA 12.8, AMD ROCm 6.4

## Installation

```
pip install realitycheck
# or
uv pip install realitycheck
```

From source:
```
git clone https://github.com/lhl/realitycheck.git
cd realitycheck
uv sync
```

## Quick Start

1. Initialize project: `rc-db init-project`
2. Set environment: `export REALITYCHECK_DATA="data/realitycheck.lance"`
3. Add claims and sources via CLI
4. Search semantically across the database

## Integrations

- Claude Code plugin with slash commands (`/reality:check`, `/reality:search`, etc.)
- Codex, Amp, OpenCode, and Pi skills
- YAML import/export
- HTML extraction utilities

## Taxonomy

Claim Types: Fact [F], Theory [T], Hypothesis [H], Prediction [P], Assumption [A], Counterfactual [C], Speculation [S], Contradiction [X]

Evidence Levels: E1 (Strong Empirical) through E6 (Unsupported)

Domain Codes: TECH, LABOR, ECON, GOV, SOC, RESOURCE, TRANS, GEO, INST, RISK, META

## Design Philosophy

Recommends a single unified knowledge base rather than topic-specific databases: "claims build on each other across domains (AI claims inform economics claims)."

## Links

- Public example KB: https://github.com/lhl/realitycheck-data
- Docs: docs/PLUGIN.md, docs/SCHEMA.md, docs/WORKFLOWS.md
- PyPI: https://pypi.org/project/realitycheck/
