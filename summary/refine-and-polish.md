---
url: https://github.com/jesserobbins/refine-and-polish
title: "Robo Refine and Polish"
author: Jesse Robbins
date_fetched: 2026-11-02
date_published: 2026-10
topics:
  - ai-code-review
  - claude-code
---

Jesse Robbins' `refine-and-polish` is a Claude Code skill (v0.2.1, MIT) that layers on top of [roborev](https://roborev.io), a continuous code-review tool, to run multi-iteration review-fix-rereview loops with multiple reviewer agents — `claude-code`, `codex`, `pi` — each backed by its own subscription. The repo is a self-hosted Claude Code plugin marketplace: two slash commands install it, and the entire product is one 531-line `SKILL.md` plus a worked-example ledger document.

The core idea is that a long review loop without bookkeeping fails in predictable ways: the same finding gets fixed twice, a fix-introduced regression slips through, and a deliberate design decision gets re-litigated every iteration because the next reviewer run can't see the prior reasoning. The skill's answer is a **private Markdown ledger** kept outside the public repo (`reviews/<pr-slug>.md` in a gitignored dir or sibling private repo). Every finding is typed — `NEW`, `PRE`, `REGRESSION`, `REPEAT`, `LOOP` — with a fine-grained taxonomy that distinguishes "my fix broke it" (REGRESSION) from "the reviewer never accepted the fix" (LOOP) from "the reviewer is re-raising a defended decision" (REPEAT). A deliberate-pushback list records findings you won't fix, with at least two written reasons, so the convergence criterion ("zero findings outside the pushback list") can actually terminate.

Notable rules: reviewers must run as separate jobs against the same pinned commit range (positional args, never relative selectors, to avoid a race where sequential `--branch` reviews resolve different ranges); a downed reviewer is recorded as an `N/A` coverage row and never revived through an API key — subscription-only is the constraint; and the "Low tail" problem (each fix creates new surface to critique, producing an unbounded drip of new Low findings) is handled by a budget-exhaustion stop rather than pretending the surface emptied. The worked example is the argument for the whole design: `codex` caught a broken slash command that `claude-code` missed entirely — with one reviewer, that bug ships.
