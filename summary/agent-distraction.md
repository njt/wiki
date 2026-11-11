---
url: https://developers.googleblog.com/agent-distraction/
title: "Agent Distraction"
author: Google Developers Blog (unattributed)
date_fetched: 2026-11-11
date_published: unknown (2026)
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

A Google Developers Blog post arguing that stale codebase artifacts — obsolete TODO(bug/...) comments, dead bug references, lagging documentation — are no longer merely human eyesores but active distractors for coding agents, which treat every TODO as a call to action during their planning and research phase.

Google quantified the effect using TODO bug references as an objective proxy: by cross-referencing every TODO an agent encounters against the linked bug's status and the user's prompt, across "hundreds of millions of individual agent steps" from thousands of engineers, they could automatically separate valid work from wasted detours. A multi-way fixed-effects regression (developer-clustered standard errors, controlling for task type, model tier, files read, prompt length, and step position) found unprompted bug detours add substantial per-session overhead, statistically significant at p<0.0001.

Two failure shapes are illustrated: the "obsolete detour" (agent investigating one bug gets sidetracked by a TODO for a bug fixed in 2023) and the "eager agent" (agent unpromptedly "fixes" code it encounters, committing scope creep the human later has to challenge). The mitigation is an agentic TODO cleaner: agents scanned the whole codebase, confirmed obsolescence by examining years of bug/file/commit history, and only then submitted deletions — thousands removed over six months with >95% human-review acceptance.

The closing argument: "agent friendly codebases" require the forgotten corners to be healthy too, and debt-cleanup projects previously deprioritised as low-value need reevaluation now that agents act on the codebase at scale.
