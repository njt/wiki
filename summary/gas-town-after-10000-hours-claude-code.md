---
url: https://simonhartcher.com/posts/2026-01-19-my-thoughts-on-gas-town-after-10000-hours-of-claude-code/
title: "My thoughts on Gas Town after 10,000 hours of Claude Code"
author: Simon Hartcher
date_fetched: 2026-05-15
date_published: 2026-01-19
topics:
  - agent-coding-workflow
---

# My thoughts on Gas Town after 10,000 hours of Claude Code

Simon Hartcher reflects on using Claude Code intensively ("it feels like a lifetime") and evaluates Steve Yegge's Gas Town multi-agent system against his preferred pair-programming workflow. He finds Gas Town impressive as a technical achievement but ultimately not for him — the loss of visibility, the slow token speed of current models, and the git-pollution of beads state make him prefer interactive pairing over agent delegation.

## Key Quotes

- "I know that there are not 10,000 hours in a year. I've been living inside Claude Code and it feels like a lifetime."
- "I feel as though I have more agency when I work this way" (pair programming with Claude Code)
- "it feels like deferring everything to agents, and I get almost no visibility of what's going on"
- "the whole process just seems really slow"
- "Beads is at the heart of Gas Town."
- "those changes pollute every PR you or an agent makes"
- "agents need contracts"
- "It should be separated from the code in my opinion."
- "Even if I am vibe engineering, I still care about the code. I still look at it."
- (Yegge, quoted by Hartcher) "I've never seen the code, and I never care to, which might give you pause."

## Key Concepts

- Gas Town: Steve Yegge's multi-agent orchestration system
- Beads: dependency-tracking tool using git for state storage, at the heart of Gas Town's task coordination
- Pair programming as Hartcher's preferred AI workflow — driver/observer mode rather than delegation
- Claude Opus 4.5 and Claude Max ($200/month) as the underlying AI infrastructure
- Agents need contracts (dependencies as a graph: A blocks B) because they lack human intuition for task ordering

## Overall Stance

Hartcher respects Gas Town as "a look into the future of low touch agentic focused workflows" and credits Yegge's engineering, but concludes it's not for him yet. His central tension: the promise of hands-off agent orchestration versus his need for visibility, involvement, and care for the code itself.
