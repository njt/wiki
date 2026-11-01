---
url: https://github.com/wack/slcr
title: "SLCR — an open standard for agent-native software requirements"
author: wack
date_fetched: 2026-11-02
date_published: 2026
topics:
  - specifications-as-the-product
  - guardrails-and-feedback-loops
---

SLCR (Sliceable, Linked, Checkable Requirements) is an open standard and a Rust reference CLI (`slcr`) that stores a project's requirements as a *graph* rather than a document. Requirements live in YAML, JSON, or TOML as sections, requirements with RFC 2119 modalities (MUST/SHOULD/MAY), acceptance criteria, and glossary terms, linked by CONTAINS, REFINES, DEPENDS_ON, and USES_TERM edges.

The standard's answer to context rot is the **slice**: `slcr slice REQ-001` prints exactly the subgraph an agent needs to implement one requirement — the requirement in full, its refinement descendants in full, its containment ancestors' bodies, its dependency closure, and the terms it uses — and nothing else. The answer to drift is **validators**: TOML files beside the code that bind an acceptance criterion to a pinned, descriptor-declared tool (a test runner, `ast-grep`, anything). `slcr check` runs every validator, combines verdicts under strong Kleene logic with the requirement refinement structure (AND/OR/XOR), and reports each requirement satisfied, unsatisfied, or unknown (a coverage gap).

Notably, SLCR dogfoods itself: the repo's own `SPEC.slcr.toml` holds the CLI's requirements, and its validators are snapshot tests. A Claude skill (`skills/slcr`) teaches coding agents the loop: read the slice, write a validator that fails, implement until it passes, run `slcr check`. The spec is the artifact agents edit; humans read Markdown rendered from it (`slcr render --expect-no-diff` fails CI if stale).
