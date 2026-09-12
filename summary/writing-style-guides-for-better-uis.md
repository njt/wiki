---
url: https://ai.statico.io/2026/07/14/writing-style-guides-for-better-uis/
title: "Writing Style Guides for Better UIs"
author: Ian Langworth
date_fetched: 2026-07-18
date_published: 2026-07-14
topics:
  - agent-coding-workflow
---

Langworth argues that UI text quality improves dramatically when you give a coding agent a writing style guide and tell it to apply the rules to every user-facing string — buttons, labels, error messages, menus — in a single pass.

He recommends the [IBM Carbon Design System's content guidelines](https://carbondesignsystem.com/guidelines/content/overview/), which emphasize everyday language, short words, and a tone that adapts to context (economical for errors, friendlier for onboarding). The Carbon writing-style source is available as raw MDX on GitHub and can be fed directly to an agent. A better approach: have the agent extract the rules once and save them as a reusable local skill.

This work was historically slow and collaborative — what Langworth calls "committee-shaped work." Decisions about labels, verb choice, and settings hierarchy once took hours or days. An LLM does a competent version in seconds, and for copy, competent-in-seconds beats perfect-in-a-week.

The practical workflow: sketch UI copy in rough language, then let the agent rewrite it against the style guide. He also mentions [The Elements of Style](https://github.com/obra/the-elements-of-style) as a Claude Code plugin derived from Strunk's 1918 text, though he hasn't tried it.
