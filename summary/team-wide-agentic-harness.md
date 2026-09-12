---
url: https://ai.statico.io/2026/07/01/team-wide-agentic-harness/
title: "Team-Wide Agentic Harness"
author: Ian Langworth
date_fetched: 2026-07-18
date_published: 2026-07-01
topics:
  - agent-coding-workflow
---

Ian Langworth describes his emerging role as a "Harness Guy" — someone who
optimizes the environment around AI coding agents so the team gets more
consistent, correct results. He runs brown-bag lunches where colleagues trade
tips and he demonstrates skills (such as one that manages a Linear queue,
generating customer-tailored release notes).

Skills are files, so they're inherently shareable and check-in-able. What's
harder to share are the *conventions* those skills depend on: sandboxing rules
that scope all agent work to a single directory, keeping dotfiles, email, and
package installation out of reach, with standard directories for worktrees,
plans, and temporary artifacts.

Langworth argues the whole thing — conventions, skills, context documents —
should live in a version-controlled "harness" directory. He draws on *The
AI-Native Startup Handbook*, which recommends a shared repository of context,
plugins, and skills across an organization. An `evergreen` docs directory
describes the company, product, and recurring procedures — the atomic blocks of
context fed into a context window before dispatching an agent.

Because skills are code, they deserve PR review: another set of eyes before
they change everyone's agent behavior. Langworth closes by wondering whether
conventions that work comfortably in one person's head will survive contact
with a full team.
