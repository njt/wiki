---
url: https://github.com/github/gh-stack
title: GitHub Stacked PRs (gh-stack)
author: github
date_fetched: 2026-08-07
---

# gh-stack

A GitHub CLI extension for managing stacked branches and pull requests. Stacked PRs break large changes into a chain of small, reviewable pull requests that build on each other. `gh stack` automates the tedious parts — creating branches, keeping them rebased, setting correct PR base branches, and navigating between layers.

Built in Go (~52K lines) as a `gh` extension, it combines a CLI (via Cobra), Bubble Tea TUIs for interactive screens (submit, modify, checkout, merge), and a local-first JSON metadata file stored in `.git/gh-stack`. Stack state is local and authoritative; the GitHub Stacks REST API mirrors it remotely. Each PR's base is the branch below it in the stack, so reviewers see only the diff for that layer.

Key commands span the full lifecycle: `init` / `add` (create), `rebase` (cascading rebase with conflict recovery), `sync` (fetch → reconcile → rebase → push → sync PR state), `submit` (push + create/update PRs via interactive TUI or `--auto`), `merge` (all-or-nothing batch merge up to a target PR), `modify` (TUI-based restructure: drop, fold, insert, reorder, rename), and navigation (`up`/`down`/`top`/`bottom`/`trunk`).

Notable design choices: optimistic concurrency via SHA-256 checksums on the stack file, platform-specific advisory locking, interface-based testability with mock GitHub and Git clients, a cascading rebase state machine with JSON-persisted recovery state, and an AI agent skill (`skills/gh-stack/SKILL.md`) so coding agents can drive stacked PR workflows. Exit codes are standardized (0–10) for scriptability.
