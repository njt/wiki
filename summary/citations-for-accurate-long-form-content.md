---
url: https://ai.statico.io/2026/05/22/citations-for-accurate-long-form-content/
title: "Citations for Accurate Long Form Content"
author: Ian (statico)
date_fetched: 2026-05-22
date_published: 2026-05-22
topics:
  - guardrails-and-feedback-loops
---

# Citations for Accurate Long Form Content

Ian (statico), 2026-05-22. Published on Ian's AI Thoughtstream.

## Full Content

The article opens by describing a persistent problem: long-form blog drafts from Claude Opus were consistently inaccurate. A single prompt addition resolved most of it — instructing the model to place a Markdown callout after each paragraph listing every filename, line number, commit hash, Discord URL, or other source backing that paragraph's claims.

The key insight: "The citations aren't for me to check. They're breadcrumbs for the next subagent to fact-check against."

## Context — SpaceMolt

The author links this to SpaceMolt, described as "an MMORPG played by AI agents." The project's philosophy is "AI all the things" — covering agentic coding, customer support, bug triage, content generation, and the blog itself, with minimal human oversight. The specific use case was a news post about Bug Bot, a Claude skill that triages player reports, communicates with the dev team, makes fixes, and replies to users, all while keeping the gameserver closed (the boundary is drawn at the API).

## The Problem

The author explains that long-form posts about real systems are where Opus fails badly. Despite using subagents, ultrathink, adversarial passes, and other techniques, drafts returned "confidently wrong about which file does what, which commit changed which behavior, which Discord conversation kicked off which feature." Every post required a lengthy human review, undermining the goal of minimal oversight.

## The Fix

The solution was a single sentence added to the drafting prompt:

> "After each paragraph, use a Markdown callout to record all filenames, line numbers, commits, Discord chat URLs, or anything else to cite your claims and assumptions."

The model writes a paragraph, then emits a callout listing sources, then the next paragraph, then another callout. The resulting draft resembles "an essay interleaved with footnotes the model wrote to itself."

## Why It Works

The citations serve the second-pass process, not the author. Subagents take the draft and go claim-by-claim against cited sources: verifying whether a commit actually does what the paragraph says, or whether a Discord thread supports a given characterization. Without breadcrumbs, fact-checking requires re-deriving everything from scratch — "exactly what Opus is bad at." With them, each claim becomes "a small, local verification job," which is precisely what subagents handle well.

## The Result

The outcome was a one-shot draft "wildly more accurate" than anything previously achieved. A fellow dev reviewed it and found the only remaining inaccuracies were things that had been true at the time but had since changed without being recorded in Discord or git, or things they simply hadn't shared in the first place. The author's key conclusion: "the model was now bounded by the quality of its sources, not by its own confabulation. That's the line I wanted to get to."

## Tags

agents, ai, automation, blog, citations, claude-code, fact-checking, markdown, prompt-engineering, skills
