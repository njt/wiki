---
url: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#writing-density
title: "Prompting Claude Fable 5.1"
author: Anthropic
date_fetched: 2026-09-04
date_published: 2026 (undated)
topics:
  - agent-coding-workflow
---

Anthropic's field guide to the behavioral differences between Claude Fable 5.1 and its predecessor, and the prompting patterns that address them. The core claim: existing Fable 5 prompts should mostly work unchanged, but a handful of deltas are worth knowing — each mapped from a symptom you'd observe to a targeted fix.

The deltas cluster into a few groups. **Effort and cost:** effort level names don't mean the same amount of thinking across models, so re-run your sweep; `medium` roughly matches Fable 5 at lower cost, and `low` is often competitive with Opus/Sonnet on cost-per-task. **Long-horizon behavior:** the model defaults to fewer user-facing progress updates, may issue tool calls one-per-turn in coding loops, and sometimes ends its turn describing what it *would* do next instead of doing it — all fixable with short nudges, several delivered as turn-scoped system messages (`clear_at: "next_user_message"`, beta). **Conversation hygiene:** thinking blocks are bound to the exact conversation that produced them, so history must stay append-only; client-side compaction needs an explicit instruction on what to preserve.

A second group concerns output style and scope. On **writing**, Fable 5.1's prose runs denser than Fable 5's — fewer paragraph breaks, longer sentences — and it's more likely to reproduce retrieved source text without marking quotations; Anthropic ships a "mannered prose" anti-pattern definition and a worked quoting example as fixes. On **scope**, it may over-deliver (fix nearby code, commit more tests than asked) or under-deliver (stop early, ask permission for work already requested), addressed by an explicit "the request is the deliverable" block. On **safety**, false positives still occur — compile-check phrasing, lesser-known languages, and base64 in tool output make them more likely.

A final cluster covers mechanics: prefer targeted edits over whole-file rewrites, leave room in `max_tokens` for thinking at `xhigh`/`max` effort, let the lead agent keep working while subagents run, and give vision work crop-and-zoom tools. Several referenced betas carry 2026-08 headers, marking this as documentation for a model generation that shipped in late 2026.
