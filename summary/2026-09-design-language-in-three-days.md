---
url: https://blog.curiosity.ai/blog/2026-09-design-language-in-three-days
title: "How we rebuilt our design language in three days with AI"
author: curiosity.ai team
date_fetched: 2026-10-10
date_published: 2026-09
topics:
  - agent-coding-workflow
  - specifications-as-the-product
---

A field report from curiosity.ai on rebuilding their whole brand system with coding agents. It started when a NYT Bauhaus piece prompted the question "what would our site look like as a Bauhaus poster?" — the next afternoon the agent had rebuilt all 67 pages. From there they ran twenty-one more full-site design directions, judged them in a replayable pairwise "showdown" game, and folded the winning style into a brand spec (`BRAND.md`) written "for a reader that has no eyes." The website, blog, docs chrome, and slide decks were then rebuilt in the brand in three days, about 200 commits, code written by Claude Code and merged through human-reviewed PRs.

The craft lessons are the substance: judge real pages with real words, not mockups; make variations cost minutes so you try the strange ones; decide pairwise with a replayable vote; write briefs as numbers rather than adjectives ("the escaped square is 20 units on a 24 unit grid, one cell out on the diagonal"); keep content fixed while the look moves; and build the checks — then check the checks. Their responsive checker reported a clean pass on every page for its whole early life without inspecting a single one, which they call "the most useful lesson in the whole project."

The "AI first" claim is architectural: everything they publish is Markdown an agent can read and write, every repo carries the instructions (`CLAUDE.md` importing `BRAND.md`, one skill per component in the blog), so "when the brand changes, one file changes, and the next session reads the new one." Ten numbered lessons close the piece, from "delete the losers" to "one source for every shared part."
