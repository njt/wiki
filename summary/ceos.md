---
url: https://github.com/bradfeld/ceos
title: CEOS — Claude + EOS
author: Brad Feld
date_fetched: 2026-08-21
date_published: 2026-08-04
topics:
  - agent-coding-workflow
---

CEOS is a Claude Code skills package that implements the Entrepreneurial Operating System (EOS) — the business-management framework from Gino Wickman's *Traction*, used by ~250,000 companies — as AI-assisted workflows. Brad Feld (Foundry Group VC, von Hippel student) built it so a leadership team can run vision, accountability, Rocks, scorecards, L10 meetings, and the rest of EOS without a SaaS subscription: all EOS data lives as markdown files with YAML frontmatter in a git repo, and the "tools" are 19 `SKILL.md` files Claude Code discovers via symlinks in `~/.claude/skills/`.

The README advertises 16 skills, but the repo actually ships 19 — `ceos-calendar`, `ceos-dashboard`, and `ceos-trends` exist but aren't in the README table (the authoring guide says "17" at one point; documentation count has drifted). The skills cover the Six Key Components: Vision (`ceos-vto`, `ceos-annual`), People (`ceos-accountability`, `ceos-people`, `ceos-quarterly`, `ceos-delegate`), Data (`ceos-scorecard`, `ceos-checkup`), Issues (`ceos-ids`), Process (`ceos-process`), and Traction (`ceos-rocks`, `ceos-l10`, `ceos-todos`, `ceos-clarity`, `ceos-kickoff`, `ceos-quarterly-planning`).

Every skill follows the same 8-section structure — frontmatter (`name`, a `description` that starts "Use when", `file-access`, `tools-used`), heading, When to Use, Context, Process, Output Format, Guardrails, and Integration Notes. The two universal guardrails are "never auto-invoke another skill" and a first-use sensitive-data warning. The load-bearing idea is a **data-ownership table**: each skill owns exactly one data directory and may only read (never write) the others, declared in an `Orchestration Principle`/`Write Principle`/`Read-Only Principle`. Orchestrator skills (`ceos-l10`, `ceos-annual`, `ceos-quarterly-planning`, `ceos-kickoff`) read broadly but write only to their own directory; `ceos-trends` is a pure read-only aggregator across all six components.

Setup is a single bash script (`setup.sh`): it discovers `skills/ceos-*/` by glob, symlinks them into `~/.claude/skills/` (idempotent), and optionally runs a guided init that prompts for company name/quarter/team, then scaffolds `data/` from templates with `sed`-based `{{placeholder}}` substitution. A hidden `.ceos` marker file at the repo root is how every skill locates the repo (search upward, then `git pull --ff-only` to sync teammates' changes).

The one real code artifact is `dashboard/build.py` (~3,300 lines): a dependency-free Python script that regex-parses the markdown data tree into a self-contained HTML dashboard (Tailwind via CDN, JSON inlined, client-side JS doing CRUD straight against the GitHub Contents API). It has a genuinely careful security posture — a `GITHUB_WRITE_TOKEN` inlined into the page is bound to a `DASHBOARD_PASSWORD`; the build *refuses* to run (rather than warn) if you set a token without a password, or a password without the `cryptography` package, so a write credential can never be published in plaintext. The password wraps the HTML in AES-256-GCM decrypted in the browser via WebCrypto. Tests (pytest, 60+ cases) include a live-fire assertion that greps the written bytes for the token rather than trusting an "encryption enabled" flag.
