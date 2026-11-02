---
url: https://github.com/jesserobbins/greenroom
title: "greenroom — a private notes repo beside every public repo"
author: Jesse Robbins
date_fetched: 2026-11-02
date_published: 2026
topics:
  - agent-coding-workflow
  - claude-code
---

greenroom is an installable Claude Code skill (also distributable via `npx skills add` to 70+ agents) that scaffolds and maintains a two-repo project layout: a public code repo and a sibling private notes repo (`docs/`, `notes/`, `drafts/`, `reviews/`, `research/`) under one non-git "wrapper" folder. The private side holds design docs, PRDs, PR review notes, and drafts under git — versioned, diffable, greppable, and reachable by a coding agent in the same session — but never pushed publicly.

The core rule is **launch at the wrapper**: `cd <project> && claude`. Every repo is then under cwd, session history lands in one bucket, and each repo's `AGENTS.md` loads lazily on first touch. Adapters bridge the two holdouts: a one-line `CLAUDE.md` containing `@AGENTS.md` for Claude Code, and `.gemini/settings.json` for Gemini CLI. A detected VS Code-family editor gets a generated multi-root `.code-workspace` with a per-project title-bar color derived from an MD5 hash of the project name.

Implementation is a single 1,839-line stdlib-only Python script (`skills/greenroom/scripts/greenroom.py`) with four subcommands: `retrofit` (wrap an existing repo), `new`, `sync`, and `collect`. Safety is the design spine: the script never pushes, never commits in the public repo, never creates a remote (it prints `gh repo create --private` commands for the user), refuses `$HOME`, the filesystem root, and standard directories as targets, honors an optional `GREENROOM_ROOT` boundary, and restores a repo if an in-place move fails. A `collect` subcommand recovers private-shaped docs already committed to public history — reading them from git at specific SHAs and copying (never rewriting) into the private repo, dry-run by default. A 2,202-line black-box smoke test exercises the reliability-critical paths.
