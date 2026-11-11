---
url: https://oreillyradar.substack.com/p/the-new-knowledge-artifacts-of-the
title: "The New Knowledge Artifacts of the AI Economy"
author: Tim O'Reilly
date_fetched: 2026-11-11
date_published: 2026-11-04
topics:
  - agent-memory-and-context
  - specifications-as-the-product
---

Tim O'Reilly's Radar essay argues that AI is displacing source code as the primary shareable knowledge artifact of software development, replacing it with what he calls "context bundles" — prompts, specs, agent skills, MCP servers, agent traces, evals, even well-formed conjectures. He grounds the argument in history: every technology revolution creates new mechanisms, new coordinating institutions, and new valuable artifacts (binaries → source repos → web pages → clicks → ...), and the institutions for the new context era are still unbuilt.

Key moves:

- **Value shifts to what's scarce.** When code is cheap to generate, the valuable thing is the context that tells the model what to generate and how to do it well. A repo may become merely a source of context for solving one problem, not a packaged solution to a bundle of problems.
- **The clothesline paradox, again.** Open source created far more value than it captured; the same will be true of whatever artifact emerges here. Christensen's Law of Conservation of Attractive Profits: when one thing commoditises, something adjacent becomes valuable.
- **A catalogue of candidate artifacts.** Spec-driven development (the spec is what is reviewed and versioned), Agent Skills as an open standard, MCP servers, agent traces ("the closest thing we have to commit history" for non-code work), evals as domain-expert knowledge, and questions/conjectures as shareable objects.
- **The trust problem.** Drawing on Cronon's *Nature's Metropolis* (Chicago grain grading, receipts, futures), he argues new artifacts invite counterfeits — and notes Nick Vincent's warning that once evals leak into training data, they measure memorisation, not capability.
- **Signals from the field.** Foo Camp sessions on "I don't want someone else's code any more"; maintainers drowning in automated contributions; risk to verified scientific libraries; and a survey of nascent institutions: Cursor Origin, Entire (Thomas Dohmke), Common Fabric, Anthropic's internal Scion VCS, Duncan Davidson's personal-context work, Sam Schillace's agent-building patterns.

His closing bet: we are at the GitHub-in-2007 moment for context — artifacts appearing, coordinating institutions a decade away. The open problem is that "we haven't yet figured out what needs to be shared."
