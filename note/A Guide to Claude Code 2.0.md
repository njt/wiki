# A Guide to Claude Code 2.0

Sankalp's field guide to Claude Code 2.0: a deep dive into sub-agents, skills, hooks, and system reminders, framed around his personal workflow and the emerging discipline of context engineering. The best single-article tour of Claude Code 2.0's feature surface written from daily practice, not documentation.

---

## Key Quotes

> "it's a little spirit/ghost that 'lives' on your computer"

Karpathy, quoted as the epigraph. Sets the tone: Claude Code as companion, not tool.

> "you can spend more time on taste refinement"

Sankalp's pitch for the "throw-away first draft" workflow. Let Claude build end-to-end, then iterate with sharper prompts. Implementation speed buys you taste bandwidth.

> "Opus 4.5 has soul"

His strongest model endorsement. Opus 4.5's intent detection and communication style make it feel like a pair programmer rather than a code generator. Contrast with Sonnet 4.5, which he says produced "a lot of slop" and "haphazard changes which would lead to bugs."

> "Claude Code is not a harness, it's a product"

Claude Code includes a harness — the scaffolding, tool calling, and prompt engineering — but packages it as a cohesive product. The distinction matters: you're not assembling components, you're using something designed.

> "Context engineering is about answering 'what configuration of context is most likely to generate our model's desired behavior?'"

His definition of the discipline. Not prompt engineering, not RAG — a broader question of what goes into the limited context window and why.

> "the art and science of curating what will go into the limited context window"

A second, complementary definition. Engineering plus taste.

> "Effective context windows are probably 50-60% or even lesser"

The practical reality: attention degrades well before the technical context limit. Everything beyond ~60% is increasingly unreliable.

> "Don't start a complicated task when you are half-way in the conversation"

Start fresh for complex work. The context has already rotted.

> "I no longer look forward to new releases because they just keep happening anyways"

A marker of how fast the tooling moves. Release fatigue as a sign of maturity.

---

## Key Themes

#claude-code #context-engineering #sub-agents #skills #hooks #workflow #model-comparison #person

### The Sub-Agent Architecture

Sankalp provides the clearest public explanation of Claude Code 2.0's sub-agent design. Five types exist: general-purpose, Explore, Plan, claude-code-guide, statusline-setup. The critical detail: **general-purpose and Plan sub-agents inherit the full parent context; Explore starts fresh.** This isn't documented prominently but has massive implications for how you use them. An Explore agent with inherited context would be bloated; a general-purpose agent without it would be blind.

His critique of Explore is sharp: the summaries it returns are "lossy compression." He prefers having Opus 4.5 read relevant files directly for better attention and cross-referencing. This is a power-user observation — the feature works, but the quality ceiling is lower than direct model attention.

### Context Engineering as Discipline

This is the article's intellectual core. Sankalp frames context engineering as distinct from prompt engineering: it's about **curating what enters the context window**, not just crafting the initial prompt. The components he identifies:

- **System reminders**: Tags injected into messages and tool results that recite objectives. Borrowed from Manus's "attention manipulation through recitation" — constantly rewriting todo lists pushes global objectives into the model's recent attention span.
- **Skills**: On-demand domain expertise loaded "like Neo in The Matrix (1999)." Solve prompt bloat by keeping specialized knowledge out of the system prompt until needed.
- **Hooks**: Bash scripts at lifecycle stages (Stop, UserPromptSubmit, etc.). Enforcement, not suggestion.
- **MCP code execution**: The insight that tool *definitions* bloat context but code *execution* doesn't. "Expose code APIs rather than tool call definitions."

The whole system is "heavily engineered" so that "your task is mainly to use your judgement and prompt it in right direction." This is the same thesis as [[Components of a Coding Agent]] — the harness matters more than the model.

### Personal Workflow

Sankalp's daily practice: Claude Code + Opus 4.5 for execution, Codex + GPT-5.2-Codex-Max for review, Cursor for reading code and manual edits. The dual-model pattern has been "pretty constant for me for probably a year." Different models catch different things during review — this is the same insight behind [[Fresh Eyes]].

His "throw-away first draft" workflow: create a branch, let Claude write end-to-end, compare against your mental model, then iterate with sharper prompts. This is a variation on [[Addy Osmani's Workflow]] (spec.md first, focused chunks) but more exploratory — closer to [[Ralph]]'s Wiggum loop with human taste as the steering mechanism.

Setup details: CLAUDE.md in all repos (kept short with "pointers to READMEs"), git worktrees, a hook that clears CLAUDE.md after 1,000 lines. The hook-as-cleanup pattern is underrated — context files rot if they grow unbounded.

---

## Critical Analysis

**What this article does better than anything else:** It explains Claude Code 2.0's architecture from the *user's* perspective, not the documentation's. The sub-agent context inheritance detail alone is worth the read — it changes how you use Explore vs. general-purpose agents and isn't obvious from the UI.

**The model tier list is useful but already dated.** Sankalp wrote this in December 2025; by May 2026 the model landscape has shifted again. The enduring insight isn't the rankings but the *pattern*: use different models for generation vs. review, and pay attention to which model "has soul" (communicates intent well) vs. which produces "slop" (surface-level changes that introduce bugs). This maps to [[HN Opus 4.5 Is Not the Normal AI Agent Experience]] — the community independently converged on the same distinction between slop-generators and pair-programmers.

**Context engineering is the term that should stick.** Sankalp didn't coin it (the concept appears earlier in [[Harness Engineering]] and [[Scaling LLMs to Larger Codebases]]) but he gives it the clearest working definition. His framing of skills, hooks, and system reminders as three mechanisms for *curating the context window* is more actionable than abstract discussions of attention. This article belongs alongside [[Agent Memory and Context]] as a foundational reference for the discipline.

**The "throw-away first draft" workflow is under-theorized.** Sankalp presents it as personal practice, but it's actually a deep insight: the first draft's value is in *showing you what you want*, not in being correct. This is the same pattern as [[Code Field]] ("resist the urge to over-specify; let the code emerge smaller than your first instinct") but approached from the opposite direction — generate big, then refine down. Both are correct for different phases.

**Gap: no discussion of failure modes.** The article is relentlessly positive about Claude Code 2.0. There's no treatment of when sub-agents go wrong, when skills conflict, or when hooks create surprising behavior. Compare with [[How Intercom Uses Claude Code]], which is honest about the "worst thing in the world or complete genius" uncertainty. Sankalp's guide is better for onboarding; Intercom's is better for production operators.

**The unspoken thesis:** "Knowing how things work can help you steer the models better." This is the through-line. The entire article is an argument that understanding the harness architecture makes you a more effective user — not because you'll hack on it, but because you'll know *what the model can see and what it can't*. This is the same argument [[Claude Code Cheat Sheet]] makes implicitly: the fine print saves hours.

---

*Sources: [[summary/my-experience-with-claude-code-20]]*
*Last updated: 2026-05-15*
