---
url: https://ai.statico.io/2026/07/14/writing-style-guides-for-better-uis/
title: "Writing Style Guides for Better UIs"
author: Ian Langworth
date_fetched: 2026-07-18
date_published: 2026-07-14
---

# Writing Style Guides for Better UIs

**Author:** Ian Langworth (Ian's AI Thoughtstream)
**Date:** 2026·07·14
**Reading time:** 2 minutes

## Summary

Langworth argues that UI quality improves dramatically when you give a coding agent a writing style guide and instruct it to apply the rules across all user-facing strings — buttons, labels, error messages, menus, and so on — in a single pass.

## Key Recommendation

His go-to resource is the [IBM Carbon Design System's content guidelines](https://carbondesignsystem.com/guidelines/content/overview/), which emphasize "everyday language, short words, and a tone that adapts to the moment (economical for errors, friendlier for onboarding)."

## Why Agents Excel Here

Langworth notes this work was traditionally slow and collaborative — "committee-shaped work." Decisions about menu labels, verb choice (Delete vs. Remove), and settings hierarchy once took hours or days. He observes that "an LLM does a competent version of it in seconds," and for copy, "competent-in-seconds beats perfect-in-a-week."

## Implementation Approach

He suggests pointing an agent at the guide and directing it to apply the style to all UI text. The Carbon writing-style source file (available as raw MDX on GitHub) can be fed directly to the agent. A better approach: have the agent extract the rules once and save them as a reusable local skill across projects.

The practical workflow: sketch UI copy in rough language, then let the agent perform a rewrite pass. Langworth notes his own publishing system works similarly, except with a guide derived from his past writing rather than a published standard.

He also mentions [The Elements of Style](https://github.com/obra/the-elements-of-style), Strunk's 1918 text available as a Claude Code plugin, though he hasn't tried it yet.

**Tags:** agents, ai, coding-agents, prompt-engineering, tooling, web
