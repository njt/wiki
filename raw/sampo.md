---
url: https://github.com/bruits/sampo
date_fetched: 2026-07-05
backfilled: true
---

Steers his mighty boat in safety, Through the perils of the whirlpool, Through the watery deeps and dangers.


Automate changelogs, versioning, and publishing—even for monorepos across multiple package registries. Currently supported ecosystems: Rust (Crates), JavaScript/TypeScript (npm), Elixir/Gleam (Hex), Python (PyPI), PHP (Packagist)... And more coming soon!

Don't know where to start? Check out Sampo's documentation or join us on Discord!

Sampo is a monorepo that contains the following crates (Rust packages):

| Name | Description | Registry | README | 
|---|---|---|---|
| `sampo` | CLI to manage changesets, versioning, and publishing | README | |
| `sampo-core` | Core logic, common types, and internal utilities | README | |
| `sampo-github-bot` | GitHub App to inspect PRs and request changesets | README | |
| `sampo-github-action` | GitHub Action to automate the release and publishing process. | README | 

The `sampo` CLI is also distributed on npm so it can be installed via `pnpm`/`npm`/`yarn`/`bun` without a Cargo toolchain:

| Name | Description | Registry | README | 
|---|---|---|---|
| `sampo` | CLI to manage changesets, versioning, and publishing | README | 

Sampo is deeply inspired by Changesets and Lerna, but made with Rust and designed for multi-ecosystems monorepos. Read more about Sampo's alternatives.

Sampo uses Knope's changesets crate to parse changeset files. Knope is another Rust project inspired by Changesets, we highly recommend checking it out!

Sampo uses and enforces Semantic Versioning (SemVer), like their standard `MAJOR.MINOR.PATCH` version format and the "Version Bumps" concepts.

Sampo is an open-source project born from Bruits, a Rust-focused collective 💛
