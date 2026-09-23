---
url: https://www.kenmuse.com/blog/removing-the-noise-for-better-prompts/
title: "Removing the Noise for Better Prompts"
author: Ken Muse
date_fetched: 2026-09-23
date_published: 2026-09-23
topics:
  - agent-coding-workflow
  - agent-memory-and-context
---

Ken Muse's short practical essay on prompt hygiene: the test for any phrase in a prompt is whether it contributes goal, context, constraint, expected output, or a verification step — and if it does none of those, it is noise. Politeness ("please"), filler ("as you know…"), and vague quality asks ("make sure everything is correct") add tokens and ambiguity without adding a definition of success.

The piece runs through a realistic bad prompt (release notes + dependency checking in one breath) and diagnoses two distinct failures: multi-goal prompts force the model to guess how unrelated tasks relate, and under-specified requests leave decisions ("higher version numbers", report vs. modify, direct vs. transitive) that the author should have made. Muse's fix is a rewrite around one outcome with the source of truth named, plus a pre-send checklist: state the outcome, keep only outcome-relevant context, define constraints, explain verification, provide paths.

He also argues for determinism over agency: recurring dependency maintenance belongs to tools like Dependabot, not to an agent reasoning about it each time. The tone is measured — removing "please" won't guarantee better answers, it just stops diluting the signal — though he cites research finding ~5% accuracy loss from excessive politeness.
