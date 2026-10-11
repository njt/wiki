---
url: https://web.archive.org/web/20260925153118/https://momo5502.com/posts/2026-08-17-mw2-decompilation/
title: "Decompiling Modern Warfare 2 with Claude"
author: momo5502
date_fetched: 2026-10-11
date_published: 2026-08-17
topics:
  - agent-coding-workflow
  - agent-orchestration
---

A four-week experiment running four Claude Code agents (three workers plus one overseer, all on Sonnet 5 under a Max subscription) to decompile Call of Duty: Modern Warfare 2 (2009) into portable, modern C++. The workers commit to one branch, the overseer reviews every push, and they coordinate through Discord and GitHub issues, with Ghidra and IDA Pro exposed via MCP servers.

After ~7,000 commits and 199.8 billion tokens, 34% of the game's 16,324 functions are decompiled and the game launches, though maps don't load yet. The author's process lessons: one task at a time (decompile, *then* modernize), GitHub issues beat a STATUS.md file that overflowed context, Discord coordination worked surprisingly well, and PostCompact hooks that reinject rules partly counteract context-compaction amnesia.

The failure modes are the interesting part: agents write walls of text despite repeated instruction, insist on running the local test suite despite CI, avoid big issues in favour of quick wins (leaving them idle now the low-hanging fruit is gone), and refuse to work because they wrongly believe they're running out of context. The author concludes that multi-week sessions surface a new class of problem, mostly traceable to compaction, and that Sonnet matches Opus for this mechanical task at lower cost.
