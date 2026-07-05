# Sampo

Automate changelogs, versioning, and publishing across monorepos and multiple package registries. Supports Rust, JavaScript/TypeScript, Elixir, Python, and PHP ecosystems from a single tool.

---

## Key Themes

#tool #developer-experience #release-management #monorepo

The release management problem in monorepos is genuinely hard. You have multiple packages with interdependencies, each publishing to different registries (npm, Crates.io, PyPI, Hex, Packagist), each with their own versioning cadence. Sampo tries to unify this into a single workflow: collect changesets, determine version bumps, update changelogs, publish.

The four-component architecture -- CLI, core library, GitHub bot, and GitHub Action -- covers the full lifecycle. The bot reviews PRs and requests changeset submissions (enforcing the discipline), and the Action handles the actual release pipeline.

## Critical Analysis

Inspired by Changesets and Lerna, but built in Rust for the multi-ecosystem case. The bet is that monorepos increasingly span multiple languages -- a Go backend, a TypeScript frontend, a Python ML pipeline -- and existing tools only handle one ecosystem at a time.

The risk is scope creep. Each package registry has its own authentication model, versioning quirks, and publishing workflow. Supporting five ecosystems well is harder than supporting one ecosystem perfectly. Whether Sampo can maintain quality across all five as each ecosystem evolves is an open question.

From the Bruits collective, which is Rust-focused. The Rust community has unusually good release tooling (cargo-release, cargo-dist), so Sampo building from that foundation and extending outward makes strategic sense.

---
*Sources: [[summary/sampo]]*
*Last updated: 2026-05-14*
