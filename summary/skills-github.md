---
url: https://github.com/jesserobbins/skills
title: "skills — Jesse Robbins' public Claude Code skills marketplace"
author: Jesse Robbins
date_fetched: 2026-11-02
date_published: 2026 (ongoing; marketplace v0.3.4)
topics:
  - claude-code
  - agent-coding-workflow
---

Jesse Robbins' public Claude Code skills marketplace: a catalog repo (`.claude-plugin/marketplace.json`) that distributes two skills as plugins, each living in its own canonical repo and mirrored here so skills.sh can index them. The through-line of both entries is **private scaffolding around public work** — keeping design docs, drafts, and review notes under git beside public code without publishing them.

**greenroom (v0.2.3)** installs a per-project layout: a non-git wrapper folder holding a `<project>-public/` repo (the stage) beside a `<project>-private/` repo (the green room). A 1,838-line Python script (`scripts/greenroom.py`) creates (`new`), wraps existing repos (`retrofit`), re-syncs (`sync`), and recovers already-public private-shaped docs (`collect`) — the latter via a path-glob rule table and private-branch-prefix heuristics, with a dry-run-then-`--apply` flow. Leak hygiene is the point: the script never pushes, never creates a remote, only *prints* `gh repo create --private` commands for the user to run, and SKILL.md drills agents to reference public artifacts by URL, never by private local path.

**roborev-refine-and-polish (v0.0.2-alpha)** is a SKILL.md-only discipline (427 lines, no code) layered on roborev's review-fix-rereview loop. It mandates a private Markdown ledger — one row per finding across all iterations and all reviewer agents (claude-code, codex, pi) — with a strict type taxonomy (NEW / REGRESSION / REPEAT / LOOP), an explicit convergence criterion, a deliberate-pushback list so defended design calls don't become infinite loops, and honest handling of downed reviewers: record as N/A, never revive with an API key, caption one-agent convergence with a coverage caveat.

Distribution is deliberately dual and independent: `/plugin marketplace add jesserobbins/skills` for Claude Code, or `npx skills add jesserobbins/<repo>` for any agent the skills CLI supports. A daily GitHub Action (`sync-mirrors.sh`) re-copies the newest release from each source repo byte-identically, with a `--check` staleness mode and a MIRRORS.md provenance record; nothing is ever pushed back upstream.
