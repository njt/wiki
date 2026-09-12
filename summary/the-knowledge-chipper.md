---
url: https://jg.gg/2026/08/04/the-knowledge-chipper/
title: The Knowledge Chipper
author: jg (jg.gg)
date: 2026-08-04
topics:
  - agent-memory-and-context
  - agent-coding-workflow
---

A practitioner's lament about the enormous waste in AI-assisted development: agents build up rich mental models of codebases across thousands of tokens of context-gathering, then nearly all that knowledge vanishes when the session ends. What remains is a commit message and whatever code comments the LLM deemed fit.

The author frames this through the lens of LLM portability — one teammate uses Codex, another uses Claude, and each starts from scratch on the same codebase. The problem becomes concrete with a story about AWS Bahrain being disrupted by war, making regional service lock-in suddenly urgent for companies that can't let AI workloads run elsewhere.

The real sting is the code review angle. A junior dev vibe-codes a massive privacy change, shuts their laptop, and goes home. The senior engineer who actually knows the privacy layer has to review it with zero context — they can't resume a session they never started. The only option is to ask *their* LLM to rebuild all the same context that was already built once. "This feels insane."

The deeper worry: as LLMs get better at producing large, nuanced code changes, the context gap between author and reviewer becomes unbridgeable. The article references Philip's work on code review in an era where AI drastically outpaces human capabilities, and frames today's PRs as "freshly minted parcels of (un-\|semi-)documented complexity."
