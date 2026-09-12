---
title: "Before Reading Code"
url: https://piechowski.io/post/git-commands-before-reading-code/
date_fetched: 2026-05-14
section: "Producing and Operating Software"
topics:
  - software-engineering-craft
---

Five git log commands that provide diagnostic insights into a codebase's health and history before examining any actual code files.

1. Code churn hotspots -- identifies most-changed files. A 2005 Microsoft Research study showed "churn-based metrics predicted defects more reliably than complexity metrics alone."
2. Contributor ranking -- maps bus factor risks. If one person authored 60%+ of commits or the top contributor disappeared six months ago, organizational risk escalates.
3. Bug cluster detection -- finds repeatedly-breaking code. Files appearing on both churn and bug hotspot lists represent highest-risk code.
4. Velocity trending -- reveals acceleration or decline. Timeline analysis connects code metrics to human factors ("that's when we lost our second senior engineer").
5. Crisis pattern detection -- counts reverts and hotfixes. Regular reverts and hotfixes suggest "unreliable tests, missing staging, or a deploy pipeline that makes rollbacks harder than they should be."

These diagnostics require minutes to run but provide substantial direction for code review priorities and organizational understanding.