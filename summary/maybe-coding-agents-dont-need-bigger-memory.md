---
title: "Maybe Coding Agents Don't Need a Bigger Memory. Maybe They Need Continuity."
url: https://oldskultxo.substack.com/p/maybe-coding-agents-dont-need-a-bigger
author: Santi (oldskultxo)
date_fetched: 2026-06-09
date_published: 2026-06-05
source: Substack (also published on dev.to)
topics:
  - misc
---

A practical reflection on why coding agents lose the thread between sessions, and why the repository itself is the right place to preserve it.

## Summary

The problem isn't memory — it's continuity. Context is what the agent has available *now*; continuity is what lets the *next* session pick up from what actually happened before. Bigger context windows, vector stores, and chat history all fail because they don't provide provenance: was this observed, validated, assumed, or contradicted?

The author argues that continuity should live as local, inspectable artifacts in the repository — not trapped inside one chat session or a proprietary tool. What agents actually need is a compact "execution surface": active work state, relevant decisions, known failures, validation expectations, and next actions — all evidence-weighted.

Through building AICTX (an open-source repo-local continuity runtime), the author discovered that **failure memory** (tracking what went wrong) and **Work State** (preserving unfinished work) were the two biggest turning points. Execution contracts provide soft guidance; guardrails should be compact and appear only at key boundaries. The hardest part is pruning — continuity should optimize for reuse, not accumulation.

## Key Quotes

> "A coding agent can have a large context window and still lose the operational thread."

> "Context is what the agent has available now. Continuity is what lets the next execution continue from what actually happened before. Those are not the same thing."

> "A memory item like 'We probably fixed the parser by changing the tokenizer' is weak. A continuity record like 'Task: fix parser edge case, Files edited: src/parser/tokenizer.py, tests/test_parser.py, Command run: pytest tests/test_parser.py, Result: passed, Known failure: full test suite still not executed, Next action: run full parser test group, Evidence quality: partial' is much stronger."

> "The mistake is treating all of these systems as the same category because they all use the word 'memory'. They are not the same category. A coding agent does not only need to know more. It needs to continue better."

> "That is where generic memory becomes weak. Not because semantic retrieval is bad, but because execution continuity needs lifecycle, timestamps, quality signals, and explicit evidence boundaries."

> "The better the continuity layer became, the less I wanted it to return."

> "If a continuity layer cannot say 'this is stale', 'this is unverified', 'this was demoted', or 'this lacks validation evidence', then it is too trusting and that is dangerous."

> "The goal is not to build a second repository made of summaries. The goal is to preserve the minimum operational state required to avoid starting from zero."

## Architecture

The proposed architecture is deliberately boring:

```
repo-local artifacts
├── active work state
├── handoffs
├── decisions
├── failure memory
├── strategy hints
├── execution summaries
├── validation expectations
├── continuity quality
└── optional repo map

lifecycle
├── resume
├── work
└── finalize

interfaces
├── CLI
├── local MCP tools
└── generated agent instructions
```

## Economics

Continuity has overhead. The value curve:
- 1–2 prompts: probably not worth it
- 3–7 prompts: break-even zone
- 7+ prompts: increasingly useful
- multi-session: strong use case
- cross-agent: very strong use case

## Reference Implementation

AICTX — open-source repo-local continuity runtime for coding agents. CLI + MCP interface, local-first, no cloud dependency.
