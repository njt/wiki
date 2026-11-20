---
url: https://github.com/starslingdev/skills
title: "StarSling Skills — Agent skills for GitHub Actions auditing"
author: StarSling (starslingdev)
date_fetched: 2026-11-20
date_published: 2026 (ongoing; active repo, latest commit 2026-11-19)
topics:
  - claude-code
  - guardrails-and-feedback-loops
---

StarSling Skills is a repository of four installable agent skills — `ci-speedup`, `ci-score`, `ci-secure`, and `sling` — distributed via `npx skills add starslingdev/skills` (the skills.sh ecosystem) for Claude Code, Codex, and other coding agents. Three of them audit a repository's GitHub Actions workflows from a local checkout: `ci-speedup` (90+ optimization patterns measured against real run history), `ci-score` (an offline pass/fail best-practice rubric), and `ci-secure` (a closed set of 10+ critical Actions attack vectors). The fourth, `sling`, drives StarSling's own `sling` CLI against their control plane for usage, billing, and run diagnostics — read-only, with state changes routed to `gh`.

What distinguishes the repo is the architecture under the skills. Each SKILL.md orchestrates a deterministic Python engine (~89K lines across the three audit skills), and the skill text is explicit that **detection, ranking, and every measured number contain no LLM**. The single non-deterministic step is a "gap-fill": when a drilled long-pole job's log matches no catalog pattern, the agent writes a labelled, log-grounded root-cause reading so no diagnosis dead-ends. Each finding ships a ready-to-paste prompt for the user's own coding agent — the tool diagnoses, the agent fixes.

The supporting artifacts are as engineered as the code: a machine-readable CI-score spec (frozen version, pure-function scorer with rounding rules), a captured command-surface JSON proving `sling` SKILL.md never routes to a nonexistent subcommand, ~100 e2e fixture repos with recorded GitHub API responses for offline replay testing, a vendored `gate.py` CI check that enforces the scan-root/engine trust split against fork-PR attackers, and an extensive agent-UX contract (structured questions, option ordering, fallback shapes) distilled from recorded live-run failures.
