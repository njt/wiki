---
url: https://codemanship.wordpress.com/2026/09/30/deterministic-when-possible-probabilistic-when-necessary-human-when-cheaper/
title: "Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper"
author: Codemanship (London-based software craft consultancy)
date_fetched: 2026-10-03
date_published: 2026-09-30
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

Codemanship's short polemic against AI-first-by-default development. The observation: teams who set out "to use AI coding agents as much as possible" with outcomes as a secondary concern — renaming classes via Claude Code when an IDE shortcut does it more reliably with a fraction of the compute, tasking Copilot to find unused code the compiler already flags, describing code to Codex in more words than the code itself. "Arguably these people have lost the plot."

The counter-example teams use the best tool for the job: IDE refactorings, linters, background compilation for detection, and plain typing when you know what you need. Agents enter only at the gap in the tooling — the Python instance-method move PyCharm can't do — where an agent succeeds ~90% of the time and beats doing it by hand. That's a *rational* dice-throw: probabilistic cost against deterministic alternatives, accepted only where the deterministic tool doesn't exist or is slower.

The takeaway is the headline rule — deterministic when possible, probabilistic when necessary, human when cheaper — and the closing point that this is supposed to be about better outcomes, "bang for the token," not agent usage maximisation. Notably the rule has three tiers, not two: the human is also a tool that gets picked when it's the cheapest reliable option.
