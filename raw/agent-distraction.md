---
url: https://developers.googleblog.com/agent-distraction/
date_fetched: 2026-11-11
---

What is easy for humans to skim past in code, agents see as a call to action. They do not know better, latching on to that which is littered across the files read during the planning and research phase of the coding journey.

As agents' agency through tools and skills grows the ability for side-quests scales proportionally. Obsolete specs, long resolved bugs, and lagging documentation no longer rest in long forgotten corners of the codebase, but rather are unintentionally ingested as context indirecting and misdirecting the next wave of coders - the agents. This indirection is not free: in the best case it only costs tokens, steps, and time - in the worst case it distracts and grounds the agent on falsehoods leading to incorrect logic and decisions during implementation.

Measuring how often agents fall into these traps across millions of monthly sessions is impossible with manual human review - we need an objective signal derivable from the code itself. Inline TODO(bug/...) comments provide the perfect proxy. TODOs tend to be left behind long after the underlying bug is resolved or abandoned, while the linked bug ticket acts as the canonical record. By cross-referencing every TODO an agent encounters against the TODO's bug status and the user's prompt, we can automatically separate valid work from wasted detours at scale.

By leveraging the distributed traces of agent interactions across thousands of engineers and over hundreds of millions of individual agent steps, we quantify the impact of TODO distractions in the codebase and how agents act on them:

Controlling for developer, codebase complexity, task type, model tier, number of files read, initial prompt length, and the step where the bug reference appeared, using a multi-way fixed-effects regression with developer-clustered standard errors, we see that unprompted bug detours add substantial overhead (all stat sig p<0.0001). Per session:

**The Obsolete Detour:** In the following trajectory we can observe how this drift plays out. During an investigation into an unrelated bug The agent views SurfaceControlTest.java and sees // TODO(bug/XXX)

This bug, while fixed in 2023, remains lingering in the codebase leading to subsequent investigation distraction across Steps with the agent digging into the bug to learn more.

**The Eager Agent:** In the trajectory the agent encountered a TODO in the bug it was investigating and decided (unprompted) to "fix" the code

The agent executed multiple Code Search and internal knowledge search queries for the bug ultimately resulting in the agent changing the ModelDownloaderV1 code. Later in the trajectory the human challenged the scope creep.

With the model when admitting it was an incorrect increase in scope:

While in both of these cases they were only trivial to minor detours - they were real distractions costing time, tokens, and human intervention.

As part of addressing this finding at scale we created agents that scrubbed the totality of our codebases, finding bug references in the code that were marked as fixed or obsolete for cleanup. That step alone however was not enough to determine if the TODOs in the codebase were load-bearing or not. The agents had to go further looking at the history of the bug, the file, and changes to the codebase, at times spanning years of history. Only in cases where the agent was able to confirm that the TODO and the intent behind the TODO was obsolete did our agent send a change for review to delete it from the codebase. As a result of this multi-layer approach, our todo cleaner has been able to remove thousands of actually obsolete references over the last 6 months exceeding 95% acceptance from human reviewers.

In recap, as we task agents on more and more ambitious problems, "agent friendly codebases" require more than just the code in the hotpath being healthy. The forgotten corners of the codebase now more than ever carry a significant tax levied against agents.

Tooling like presubmit analyzers, entire corpus scanners, and automated coding agents to remediate dead-todos offer out of the box ready to scale solutions to address the tax enabling a healthier and distraction free codebase for agents to thrive.

Projects that once were on the backburner to cleanup lingering debt but deprioritized due to perceived low value need to be reevaluated with the new agents acting at scale on your codebase.
