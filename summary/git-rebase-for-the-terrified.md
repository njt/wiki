---
url: https://www.brethorsting.com/blog/2026/01/git-rebase-for-the-terrified/
title: Git Rebase for the Terrified
author: Aaron Brethorst
date_fetched: 2026-05-15
date_published: 2026-01-07
tags: [git, rebase, version-control, workflow, open-source]
topics:
  - software-engineering-craft
---

# Git Rebase for the Terrified

Aaron Brethorst, a maintainer of OneBusAway projects, writes to demystify git rebase for contributors who dread it. He argues that the fear is overblown because the absolute worst outcome — deleting your local clone — is trivial to recover from, since remote repos remain untouched.

## Why Maintainers Want Rebases

Feature branches grow stale as `main` advances. Merge commits produce "messy history with interleaved commits." Rebasing replays your commits atop current `main`, yielding "a clean, linear history" that simplifies review and bug hunting via bisect.

## Step-by-Step Commands

1. **Check remotes** — `git remote -v` to see what's configured.
2. **Add upstream** if missing: `git remote add upstream <URL>`.
3. **Fetch upstream changes:** `git fetch upstream`.
4. **Switch to your branch:** `git checkout your-branch-name`.
5. **Backup current work:** `git push origin your-branch-name` as a safety net.
6. **Execute the rebase:** `git rebase upstream/main`.

## Conflict Markers Explained

Standard three-section conflict block:
- **HEAD side** (top) = "your code from the commit being replayed"
- **Incoming side** (bottom, after `=======`) = "the code from main that conflicts with yours"

Brethorst endorses VS Code's conflict UI as "the clearest I've found."

## Tricky Conflict Strategies

1. **Squash first** — combine commits before rebasing to reduce conflict opportunities.
2. **Abort & pivot** — if divergence is too severe, start a fresh branch from main and reapply changes manually.
3. **Enable `git rerere`** — `git config --global rerere.enabled true` lets Git auto-reapply conflict resolutions you've made before.

## After Resolution

- Stage resolved files: `git add <file>`
- Continue rebase: `git rebase --continue`
- Abort if needed: `git rebase --abort`

## Validation

Check commit list with `git log --oneline upstream/main..HEAD` to confirm only your commits remain ahead of main. Then build and run tests.

## Force Pushing

Recommends `git push --force-with-lease` over `--force` because it "will fail if someone else has pushed to your branch since you last fetched." Warns: "Never force push to main or any shared branch."

## Nuclear Recovery Option

Push any salvageable work to your remote fork (possibly on a temp branch), delete the local clone, reclone from fork, re-add upstream, and restart the rebase. "This has never failed me."

## Ethical Boundary

Rebasing rewrites history, which "is fine for feature branches that only you are working on" but problematic for shared branches others have based work on.

## Key Quotes

- "the worst case scenario for a rebase gone wrong is that you delete your local clone and start over"
- "A merge commit can combine them, but it creates a messy history with interleaved commits"
- "Your job is to decide what the final code should look like"
- "Rebasing rewrites commit history. This is fine for feature branches that only you are working on"
