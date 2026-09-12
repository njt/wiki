---
url: https://github.com/nektos/act
title: "nektos/act — Run GitHub Actions Locally"
author: Casey Lee (cplee) and contributors
date_fetched: 2026-07-08
date_published: 2019-02-27
topics:
  - developer-tools
---

`act` is a Go CLI tool (~23K lines) that runs GitHub Actions workflows locally using
Docker. It parses `.github/workflows/*.yml`, builds a dependency-graph execution plan
via topological sort on `needs` declarations, and runs each job in Docker containers —
letting developers test CI/CD pipelines without pushing to GitHub.

The architecture is a four-stage pipeline: CLI flag parsing (90+ flags via Cobra) →
workflow planning (YAML → dependency DAG) → runner (matrix expansion, context
creation) → job executor (container lifecycle + step execution). The core abstraction
is a functional Executor pattern — `func(ctx context.Context) error` — composed with
`.Then()`, `.Finally()`, `.If()`, and parallel/sequential combinators. The entire job
pipeline is built by chaining these closures, with `ctx` carrying logging, cancellation,
and error state.

Expression evaluation (`${{ }}`) is YAML-AST-aware rather than string-substitution
based. The evaluator walks the `yaml.Node` tree, mutates nodes in place, and handles
GitHub's undocumented features (insert directives in mapping keys, sequence flattening).
Type coercion matches GitHub Actions' loose typing: `""` coerces to `0`, `"true"` to `1`.
The parser comes from `rhysd/actionlint`, but `act` provides its own interpreter.

Containers are kept alive with `tail -f /dev/null` as entrypoint, with each step run via
`docker exec`. This provides filesystem isolation, environment consistency with GitHub's
runner images, and service-container networking. A host-environment backend exists for
`self-hosted` runners. Remote actions are cached in an embedded BoltDB store keyed by
content hash, with an offline mode available.

`act` prioritizes fidelity over performance — it handles the full expression language,
workflow command protocol, composite actions, matrix builds (Cartesian product
expansion), and post-step cleanup in reverse order. The main gap vs. real GitHub
runners is the absence of job registration and result reporting to GitHub's API.
