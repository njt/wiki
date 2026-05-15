---
title: "Compound Engineering"
url: https://every.to/guides/compound-engineering
date_fetched: 2026-05-14
fetched_via: "web.archive.org (https://web.archive.org/web/20260417172052/https://every.to/guides/compound-engineering)"
section: "AI-enhanced Coding"
---

# Compound Engineering: The AI-Native Engineering Philosophy

By Kieran Klaassen, Every.to Guides, January 17, 2026.

## The Philosophy

The core philosophy of compound engineering is that each unit of engineering work should make subsequent units easier -- not harder. Most codebases get harder to work with over time because each feature you add injects more complexity. After 10 years, teams spend more time fighting their system than building on it because each new feature is a negotiation with the old ones. Over time, the codebase becomes harder to understand, harder to modify, and harder to trust.

Compound engineering flips this on its head. Instead of features adding complexity and fragility, they teach the system new capabilities. Bug fixes eliminate entire categories of future bugs. When they are codified, patterns become tools for future work. Over time, the codebase becomes easier to understand, easier to modify, and easier to trust.

## The Main Loop

Every runs five products -- Cora, Monologue, Sparkle, Spiral, and their website Every.to -- with primarily single-person engineering teams. The system that makes this possible is a four-step loop:

**Plan -> Work -> Review -> Compound -> Repeat**

The first three steps -- plan, work, and review -- should be familiar to any developer. It's the fourth step that separates compound engineering from other engineering. This is where the gains accumulate. Skip it, and you've done traditional engineering with AI assistance.

The plan and review steps should comprise 80 percent of an engineer's time, and work and compound the other 20 percent. In other words, most thinking happens before and after the code gets written.

### 1. Plan

Planning transforms an idea into a blueprint, and better plans produce better results:
- Understand the requirement. What's being built? Why? What constraints exist?
- Research the codebase. How does similar functionality work? What patterns exist?
- Research externally. What do the framework docs say? What are the established best practices?
- Design the solution. What's the approach? Which files need changes?
- Validate the plan. Does this hold together? Is it complete?

### 2. Work

Execution follows the plan. The agent implements while the developer monitors:
- Set up isolation. Git worktrees or branches keep work separate.
- Execute the plan. The agent implements step by step.
- Run validations. Run tests, linting, and type checking after each change.
- Track progress. Check what work has been done, and what remains.
- Handle issues. When something breaks, adapt the plan.
- If you trust the plan, there's no need to watch every line of code.

### 3. Review (Assess)

This step catches issues before they ship. More importantly, it captures learnings for the next cycle:
- Have multiple agents review the output. Multiple specialized reviewers examine the code in parallel.
- Prioritize findings. Mark findings as P1 (must fix), P2 (should fix), or P3 (nice to fix).
- Resolve findings. The agent fixes issues based on review feedback.
- Validate fixes. Confirm fixes are correct and complete.
- Capture patterns. Document what went wrong to prevent recurrence.

### 4. Compound (The Most Important Step)

Traditional development stops at step three, but the compound step is where the gains are to be made. The first three steps produce a feature. The fourth step produces a system that builds features better each time:
- Capture the solution. Ask yourself: What worked? What didn't? What's the reusable insight?
- Make it findable. Add YAML frontmatter with metadata, tags, and categories for retrieval.
- Update the system. Add new patterns into CLAUDE.md.
- Create new agents when warranted.
- Verify the learning. Ask yourself: Would the system catch this automatically next time?

## The Plugin

The compound engineering workflow ships as a Claude Code plugin. What's in the box:
- 26 specialized agents, each trained for a specific job
- 23 workflow commands, including the main loop plus utilities
- 13 skills providing domain expertise

Installation: `claude /plugin marketplace add https://github.com/EveryInc/every-marketplace` then `claude /plugin install compound-engineering`

### Core Commands

- `/workflows:brainstorm` -- When requirements are fuzzy, brainstorm what to build. Runs lightweight repo research, asks clarifying questions, captures decisions.
- `/workflows:plan` -- Spawns three parallel research agents (repo-research-analyst, framework-docs-researcher, best-practices-researcher), then spec-flow-analyzer. Produces structured plan with affected files and implementation steps. Enable ultrathink mode for 40+ parallel research agents.
- `/workflows:work` -- Four phases: quick start (creates git worktree), execute (implements with progress tracking), quality check (5+ reviewer agents), ship it (linting, creates PR).
- `/workflows:review` -- Spawns 14+ specialized agents in parallel: security-sentinel, performance-oracle, data-integrity-guardian, architecture-strategist, pattern-recognition-specialist, code-simplicity-reviewer, and framework-specific reviewers (DHH-rails, Kieran-rails, TypeScript, Python). Returns combined, prioritized list.
- `/workflows:compound` -- Spawns six parallel subagents: context analyzer, solution extractor, related docs finder, prevention strategist, category classifier, documentation writer. Creates searchable markdown with YAML frontmatter.
- `/lfg` -- Chains the full pipeline: plan -> deepen-plan -> work -> review -> resolve findings -> browser tests -> feature video -> compound. Spawns 50+ agents across all stages.

## Beliefs to Let Go

1. "The code must be written by hand" -- The requirement is to write good code. Who types doesn't matter.
2. "Every line must be manually reviewed" -- If you don't trust the results, fix the system, instead of compensating by doing everything yourself.
3. "Solutions must originate from the engineer" -- The engineer's job becomes to add taste -- knowing which solution fits this codebase, this team, and this context.
4. "Code is the primary artifact" -- A system that produces code is more valuable than any individual piece of code.
5. "Writing code is the core job function" -- A developer's job is to ship value. Effective compound engineers write less code and ship more.
6. "First attempts should be good" -- First attempts have a 95% garbage rate. Second attempts are still 50%. Focus on iterating fast enough that your third attempt lands in less time than attempt one.
7. "Code is self-expression" -- The code was never really yours. Letting go of code as self-expression is liberating.
8. "More typing equals more learning" -- Understanding matters more than muscle memory. The developer who reviews 10 AI implementations understands more patterns than the one who hand-typed two.

## Beliefs to Adopt

- **Extract your taste into the system.** Write preferences in CLAUDE.md so the agent reads them every session. Build specialized agents and skills that reflect your taste.
- **The 50/50 rule.** Allocate 50% of engineering time to building features, 50% to improving the system. An hour spent creating a review agent saves 10 hours of review over the next year.
- **Trust the process, build safety nets.** "When you feel as if you can't trust the output, don't compensate by switching to manually reviewing the code. Add a system that makes that step trustworthy, such as creating a review agent that flags issues."
- **Make your environment agent-native.** If a developer can see or do something, the agent should too.
- **Parallelization is your friend.** The new bottleneck is compute, not human attention.
- **Plans are the new code.** The plan document is now the most important thing you produce.

## The Five Stages of AI Adoption

- **Stage 0: Manual development** -- Writing code line by line without AI.
- **Stage 1: Chat-based assistance** -- Using AI as a smart reference tool, copy-pasting snippets.
- **Stage 2: Agentic tools with line-by-line review** -- Allowing AI to make changes, but approving every action. Most developers plateau here.
- **Stage 3: Plan-first, PR-only review** -- Collaborate on a detailed plan, then step away and let the AI implement. Review at PR level. Compound engineering begins here.
- **Stage 4: Idea to PR** -- Provide an idea, agent handles everything. Involvement shrinks to ideation, PR review, and merge.
- **Stage 5: Parallel cloud execution** -- Execution moves to cloud, multiple agents work independently on different features. You direct agents from anywhere.

## Three Questions for Reviewing AI Output

1. "What was the hardest decision you made here?" -- Forces the AI to reveal tricky parts.
2. "What alternatives did you reject, and why?" -- Shows options considered, catches bad choices.
3. "What are you least confident about?" -- Gets the AI to admit where it might be wrong.

## Best Practices

### Agent-Native Architecture
Give the agent the same capabilities you have. Every capability you withhold becomes a task you do yourself. Progressive levels: basic development (file access, tests, git) -> full local (browser, logs, PRs) -> production visibility (read-only logs, error tracking) -> full integration (ticket systems, deployment, external services).

### Skip Permissions
`--dangerously-skip-permissions` for compound engineering at stage 3+. Use when you trust the process, are in a safe environment, and want velocity. Always work in branches, have tests, review the PR. `alias cc='claude --dangerously-skip-permissions'`

### Design Workflow
Create throwaway "baby app" prototypes for design iteration. The "figma-design-sync" agent pulls designs from Figma, compares to what's built, and fixes differences. Codify design taste into skill files.

### Vibe Coding
Skip the ladder, go to Stage 4. Describe what you want, let agents build it. Optimal split: vibe code to discover what you want, then spec to build it properly.

### Team Collaboration
- Plan approval requires explicit sign-off.
- PR ownership stays with the person who initiated the work.
- Human reviewers focus on intent, not implementation (AI reviews handle syntax, security, performance, style).
- Async by default -- plans can be reviewed without meetings.

### The 50/50 Rule Applied
Previously 80/20 (planning+review vs work+compound). For broader responsibilities: 50% building features, 50% improving the system.
