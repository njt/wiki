---
url: https://dpc.pw/posts/i-dont-want-your-prs-anymore/
title: "I don't want your PRs anymore"
author: DPC (dpc.pw)
date_fetched: 2026-05-15
date_published: 2026-04-06
tags: [dev, ai, open-source, pull-requests, llm]
---

# I don't want your PRs anymore

## Why I don't want to merge your PR

Three core concerns with outside PRs:

1. **Security risk** — "I always have to assume that you might be trying to sneak in something malicious along with your changes"
2. **Subjective style differences** — formatting, dependencies, and approach preferences vary between people
3. **Coordination overhead** — review cycles, CI waits, merge conflicts, and timezone synchronization issues

With LLMs, these tradeoffs shift: no security worry about AI-generated code, style guidelines can be codified once, and iteration happens at the maintainer's own pace.

## The nature of software development has shifted

Source code is "an intermediate formalized layer between ideas in the developer's head and instructions for the computer." The real bottlenecks now are:

- **Understanding** existing code
- **Designing** the right architecture
- **Reviewing** output for correctness

"the code in your PR doesn't help me much with any of these"

## How can you help instead

Six alternative forms of contribution:

1. **Give feedback** — Users can report what works and what doesn't, since maintainers often lack time to use their own software thoroughly
2. **Discuss ideas** — Sharing diverse perspectives helps shape what to build and how
3. **Report & investigate bugs** — "A good bug report is 3/4 of the bug itself being fixed" — well-described reproduction steps are highly valued
4. **Prototype changes** — Reference PRs *and* the prompts used to generate them are welcome. "A quick glance at code implementing something can still be helpful, even if I don't end up merging it." Sharing the "source" (prompt) lets the maintainer reuse and refine
5. **Review code & point out problems** — Extra review eyes help since the author is bottlenecked on reviewing
6. **Fork & report back** — "Just fork. Add support for your own use case, do things your way, ask neither for permission nor forgiveness." This saves the maintainer time on consensus-building and multi-use-case design

## Central thesis

LLMs have made code generation cheap and low-risk for maintainers, while human-contributed PRs carry trust costs, style friction, and synchronization overhead that now outweigh their benefits. The author isn't rejecting help — they're redirecting it toward higher-leverage activities: feedback, design discussion, debugging, prototyping with prompts, code review, and independent forking.
