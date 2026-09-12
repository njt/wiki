# Before Reading Code

Ally Piechowski's five git commands to run before you read a single line of source code. Git archaeology as diagnostic: find the hotspots, the bus factor, the firefighting frequency, and the velocity trends. Minutes of work that save hours of misdirected reading.

---

## Key Quotes

> "Churn-based metrics predicted defects more reliably than complexity metrics alone." (2005 Microsoft Research study)

> "The people who built this system aren't the people maintaining it."

> "That's when we lost our second senior engineer."

## Key Themes

#observability #devtools #simplicity

Five commands, each revealing something different:

1. **Churn hotspots** -- most-changed files. High churn correlates with defects better than complexity metrics.
2. **Contributor ranking** -- bus factor. If one person authored 60%+ of commits, or the top contributor vanished six months ago, you have organizational risk.
3. **Bug clusters** -- files that appear on both the churn and bug-fix lists are the highest-risk code.
4. **Velocity trending** -- commit frequency over time. Acceleration, decline, or the telltale cliff of a key departure.
5. **Crisis patterns** -- reverts and hotfixes. Frequent reverts signal "unreliable tests, missing staging, or a deploy pipeline that makes rollbacks harder than they should be."

The insight that makes this powerful: code health is a team metric, not a code metric. The timeline analysis connects git data to human events -- departures, reorgs, crunch periods. You're reading the organization through its commits.

This pairs well with [[Nobody Knows How Large Software Projects Work]] -- if nobody can explain the system, at least you can instrument its history. And the churn-as-risk-signal idea connects to the operational awareness in [[The Future of Software Engineering is SRE]].

## Critical Analysis

Elegant and immediately actionable. The commands themselves aren't novel (git log archaeology is well-trodden), but the framing as a diagnostic protocol -- run these five before you read anything -- is. The Microsoft Research citation adds credibility to what could otherwise feel like folk wisdom.

Limitation: this only works for repos with meaningful commit history. Squash-merge workflows, monorepos with noisy commit logs, and repos where "git blame" has been rewritten make the signals unreliable. Would benefit from noting when the approach breaks down.

---
*Sources: [[summary/before-reading-code]]*
*Last updated: 2026-05-14*