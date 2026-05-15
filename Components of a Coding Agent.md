# Components of a Coding Agent

Sebastian Raschka's anatomy of what makes coding agents (Claude Code, Codex, etc.) work. The key insight: "A lot of apparent 'model quality' is really context quality." The harness -- the software wrapping the LLM -- matters more than the model itself. He identifies six core components: live repo context, prompt caching, tool access/validation, context reduction, structured session memory, and bounded subagents.

---

## Key Quotes

> "A lot of apparent 'model quality' is really context quality."

> "The harness can often be the distinguishing factor that makes one LLM work better than another."

> "Coding work is only partly about next-token generation. A lot of it is about repo navigation, search, function lookup, diff application, test execution, error inspection."

## Key Themes

#agent-architecture #context-management #coding-agents #harness

This is the best available decomposition of what a coding agent actually *is* under the hood. The six components map cleanly to real engineering decisions:

1. **Live repo context** -- gather workspace state before working, not after failing
2. **Prompt caching** -- keep stable prefixes stable, only update what changes
3. **Tool validation** -- constrain the model's freedom to improve usability
4. **Context reduction** -- clip, deduplicate, compress to fight token bloat
5. **Session memory** -- full transcript plus distilled working memory
6. **Bounded subagents** -- delegate with inherited context but tighter restrictions

The "context quality > model quality" insight connects directly to [[Three Tier Memory]], which provides a concrete architecture for the context management problem Raschka identifies. It also explains why [[claude-code-config (Trail of Bits)]] invests so heavily in configuring the harness rather than optimizing prompts.

## Critical Analysis

This is a clear, well-structured technical overview that correctly identifies the harness as the real differentiator. What's missing is the failure mode analysis -- Raschka explains how the components work when they work, but not what happens when context reduction loses important information, or when session memory drifts from reality. The pieces by [[Cognitive Debt]] and [[Slowing the Fuck Down]] fill that gap by describing what goes wrong when the system's abstractions leak. Still, this is essential reading for anyone building or configuring a coding agent.

---
*Sources: [[raw/components-of-a-coding-agent]]*
*Last updated: 2026-05-14*
