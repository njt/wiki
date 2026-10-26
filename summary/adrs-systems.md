---
url: https://adrs.systems/
title: "ADRs4AI — the record does the remembering"
author: Jérémie Lumbroso (via ADRs4AI)
date_fetched: 2026-10-26
date_published: unknown
topics:
  - specifications-as-the-product
  - agent-memory-and-context
---

The ADRs4AI landing page pitches a method, not a product: make the thinking between human and AI the durable artifact, so it compounds instead of evaporating. Concretely, a project gets a git-tracked record of Architecture Decision Records, each staking an open question (`### QST:`) that any AI can answer with options, a recommendation, rationale, stated confidence, and a falsifier; the human rules with one line — accept, override, or anything in any shape. The founding mechanic is "one seed, one question, one ruling": dump messy thinking into a seed, the AI chunks it into ADRs, every open question is one grep away, and the record begins before the building does.

The page is candid about what it is not: not transcript archival, not vector-database memory, not an orchestration platform, and not a claim of AI persistent identity — "the record holds, minds change, and the work resumes either way." The loop is deliberately attributed: the AI carries the reasoning, the human spends one judgment, and the record keeps the why "for every mind that comes later." Optional later chapters add a "crew" (several agent seats, wake-on-brief) so the human stops being the message bus.

Tooling is minimal and open: a GitHub template repo, an editor extension with an Open Questions view, an MCP server, and the humble interface that the founder rules from his phone. The showcase example is from the method's own live project (`frontier-model-observatory`), verbatim — including a bitemporal-hybrid ruling on how corrections should preserve what was previously believed.
