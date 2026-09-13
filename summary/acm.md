---
url: https://github.com/roblarsen/ACM
title: "Assumptions & Constraints Manifest (ACM)"
author: Rob Larsen
date_fetched: 2026-09-13
date_published: 2026-08-24
topics:
  - guardrails-and-feedback-loops
  - ai-code-review
---

# Assumptions & Constraints Manifest (ACM)

ACM is a proposed open engineering standard (spec v1.1.0-draft, MIT, by Rob Larsen) for forcing code changes — especially AI-generated ones — to declare their silent operational assumptions at the pull-request boundary. Its diagnosis: LLMs generate Day-1 code almost for free but embed unstated runtime assumptions (process-local state on multi-pod clusters, non-atomic dual writes, missing timeout budgets), so reviewers burn their time reverse-engineering what the model *assumed* instead of reviewing intent.

A manifest is a YAML frontmatter block plus four mandatory markdown sections, wrapped in `<!-- ACM-START -->` / `<!-- ACM-END -->` delimiters inside the PR body or a `.acm.md` file. The v1.1 "typed contract model" replaces flat booleans with five statuses — `guaranteed`, `conditional`, `unsupported`, `not_applicable`, `unknown` — applied to five mandatory contracts (idempotency, concurrency_safety, horizontal_scalability, strict_ordering, data_loss_safety) plus four distributed primitives (consistency_model, delivery_semantics, degradation_mode, backpressure_strategy). The key rule is evidence anchoring: a `guaranteed` claim must cite a concrete mechanism (a lock, an index, a test), an `unsupported` claim must declare its `failure_mode`, and CI "risk floors" make certain combinations illegal — e.g. `risk_level: low` is forbidden when data-loss or concurrency safety is `unsupported`/`unknown`.

The repo ships the RFC spec, JSON Schema and OpenAPI schemas, a small TypeScript CLI (`tooling/acm-cli`, commander + zod + yaml) that validates manifests end to end, a `.cursorrules` prompt that instructs Cursor/Copilot agents to emit code and manifest together ("dual-output generation"), a PR template with a reviewer sign-off checklist, a GitHub Actions governance gate that posts an idempotent audit comment and blocks failing PRs, and three deliberately broken example manifests (Express checkout saga, token revocation, Kafka consumer) whose honest `unsupported` declarations the CLI's own tests verify.

The whitepaper positions ACM against the existing formalisms: ADRs cover macro strategy, OpenAPI covers I/O shape, Design by Contract covers in-process invariants — ACM claims the "change horizon", the PR diff, where none of them look. It is Design by Contract elevated from the method call to the distributed PR boundary.
