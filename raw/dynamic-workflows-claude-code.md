---
url: https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code
title: "A Harness for Every Task: Dynamic Workflows in Claude Code"
author: Thariq Shihipar and Sid Bidasaria
date_fetched: 2026-07-29
date_published: 2026-06-02
site: Anthropic Blog (claude.com)
---

# A Harness for Every Task: Dynamic Workflows in Claude Code

**Authors:** Thariq Shihipar and Sid Bidasaria, members of technical staff at Anthropic working on Claude Code

**Published:** June 2, 2026

**Category:** Claude Code

## Opening

The article announces the release of "dynamic workflows" in Claude Code. As the authors explain, Claude can now author its own harness on the fly, custom-built for whatever task is at hand.

The default Claude Code harness is built for coding, which the authors note is useful for many tasks since "many tasks resemble coding tasks." However, certain task classes previously required custom harnesses built on top of Claude Code — things like research, security analysis, agent teams, or code review.

Workflows let users "dynamically create harnesses built on top of Claude Code" so Claude can solve those problems more natively. They can also be shared and reused.

## Example Prompts

The article offers several illustrative prompts, including:

- A test that fails "maybe 1 in 50 runs" — spawning a workflow to reproduce it, form competing theories about the race condition, and stopping only when evidence eliminates all but one theory.
- Mining the last 50 sessions for recurring corrections and turning them into `CLAUDE.md` rules.
- Digging through six months of Slack incidents to find recurring root causes without filed tickets.
- Having a business plan critiqued by agents playing investor, customer, and competitor perspectives.
- Ranking 80 resumes for a backend role, double-checking the top ten, and conducting an interview.
- Brainstorming CLI tool names via a tournament to pick the top three.
- Renaming a `User` model to `Account` everywhere.
- Verifying every technical claim in a blog post draft against the codebase.

## How Dynamic Workflows Work

Dynamic workflows execute a JavaScript file with special functions that spawn and coordinate subagents. They also include standard JavaScript functions (JSON, Math, Array) for data processing.

A key capability: workflows "can decide which models an agent uses and whether subagents are run in their own worktree," giving Claude the ability to choose intelligence level and isolation needed.

If a workflow is interrupted (by user action or terminal quit), resuming the session lets the workflow pick up where it left off.

## Why Dynamic Workflows

The default Claude Code harness handles planning and execution in one context window, which works for many coding tasks but "can break down over long-running, massively parallel, highly structured and/or adversarial tasks."

The article identifies three failure modes that emerge over long context windows:

- **Agentic laziness** — Claude stops before finishing complex multi-part tasks and declares partial progress complete.
- **Self-preferential bias** — The model prefers its own results, especially when verifying or judging them.
- **Goal drift** — Gradual loss of fidelity to the original objective across many turns, especially after compaction, where "each summarization step is lossy."

Creating a workflow combats these by orchestrating separate subagents "with their own context windows and focused, isolated goals."

## Dynamic vs Static Workflows

Static workflows (using the Claude Agent SDK or `claude -p`) must work for all edge cases, making them more generic. With Claude Opus 4.8 and dynamic workflows, "Claude is now intelligent enough to write a custom harness tailor-made for your use case."

## Helpful Patterns

The article presents several composable workflow patterns Claude might use:

1. **Classify-and-act** — A classifier agent decides the task type and routes to different agents or behavior.

2. **Fan-out-and-synthesize** — Split a task into many smaller steps, run an agent on each, then synthesize results. The synthesize step acts as a barrier, waiting for all fan-out agents and merging structured outputs.

3. **Adversarial verification** — For each spawned agent, run a separate agent to adversarially verify its output against a rubric or criteria.

4. **Generate-and-filter** — Generate ideas, then filter by rubric or verification, deduplicate, and return only the highest quality results.

5. **Tournament** — Spawn N agents attempting the same task with different approaches. A judging agent evaluates results pairwise until a winner emerges.

6. **Loop until done** — For tasks with unknown workload, loop spawning agents until a stop condition is met (no new findings, no more errors).

## Use Cases

**Migrations and refactors:** The article notes Bun "was rewritten from Zig to Rust using workflows." The approach: break work into steps (callsites, failing tests, modules), spin off subagents in worktrees for each fix, have adversarially reviewing agents, then merge.

**Deep research:** A `/deep-research` skill inside Claude Code uses dynamic workflows — "it fans-out web searches, fetches sources, adversarially verifies their claims, and synthesizes a cited report."

**Deep verification:** For reports needing every factual claim checked, generate a workflow where one agent identifies all claims, then subagents check each in detail, with a verification agent auditing the source agent's quality.

**Sorting:** For qualitative sorting of large lists (e.g., support tickets by severity), the article recommends tournaments, pairwise-comparison agents, or bucket-ranking in parallel rather than evaluating 1000+ rows in one prompt.

**Memory and rule adherence:** Create workflows with verifier agents — one per rule. Or mine recent sessions for recurring corrections, cluster them with parallel agents, adversarially verify candidates, and distill survivors into `CLAUDE.md`.

**Root-cause investigation:** Spin up agents generating hypotheses from disjoint evidence sources (logs, files, data separately), then have each hypothesis face "a panel of verifiers and refuters."

**Triaging at scale:** A triage workflow classifies each item, dedupes against tracked items, and takes action (attempting the fix or escalating). A useful pattern is "quarantine" — barring agents reading untrusted public content from taking high-privilege actions.

**Exploration and taste:** For design or naming tasks, have Claude explore solutions and give a review agent a rubric. The task completes when criteria are met.

**Evals:** Spin off separate agents in worktrees, then comparison agents to grade outputs against a rubric.

**Model and intelligence routing:** A classifier agent decides which model to use based on expected complexity, after researching the task shape.

## When Not to Use Dynamic Workflows

Dynamic workflows "are not needed for every task and may end up using significantly more tokens." For regular coding tasks, the article advises asking whether it "really needs more compute." Most traditional coding tasks don't need a panel of five reviewers.

The multi-agent vs single-agent decision follows similar logic — "parallelism and specialization have to earn their coordination cost."

## Tips for Building Dynamic Workflows

**Prompting:** Detailed prompting using the specific patterns described creates the best results. Users can prompt for a "quick workflow" for smaller tasks.

**Combine with `/goal` and `/loop`:** For repeatable workflows (triage, research, verification), pair with `/loop` at regular intervals and `/goal` for hard completion requirements.

**Token usage budgets:** Users can "set explicit token usage budgets" by prompting with a budget like "use 10k tokens."

**Saving and sharing:** Save workflows by pressing "s" in the workflow menu. Check them into `~/.claude/workflows` or distribute them via a skill. To share via a skill, place JavaScript workflow files in the skill folder and reference them in SKILL.md. For flexibility, users may want to prompt Claude to treat workflows as templates rather than verbatim scripts.

## Conclusion

The article frames workflows "as a starting point to explore new ways to use Claude to help accomplish your tasks," noting there is still much to discover.

It references two related pieces: the three "harness design patterns" for building with Claude, and guidance on principles for what belongs in a harness.
