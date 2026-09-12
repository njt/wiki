---
url: https://gist.github.com/a80e60a6ff03bd5a853c91b130d22bd0
title: "Matt Pocock on AI-Assisted Development — The Smart Zone and AFK Pipeline"
author: Matt Pocock
date_fetched: 2026-07-04
date_published: 2026-06-01
source_type: ytx-gist
gist_id: a80e60a6ff03bd5a853c91b130d22bd0
topics:
  - agent-coding-workflow
---

# Matt Pocock on AI-Assisted Development — The Smart Zone and AFK Pipeline

YouTube video transcribed via ytx. Speaker: Matt Pocock (github.com/mattpoc), known for Total TypeScript, now building AI-assisted development tooling.

## Summary

**Key points**

- LLMs have a "smart zone" (roughly the first 100k tokens) and a "dumb zone" where attention relationships degrade quadratically. Tasks must be sized to stay inside the smart zone.
- LLMs are like the protagonist from _Memento_—they reset to a clean base state when context is cleared. Compacting (summarizing history) is worse than clearing because clearing gives a predictable, repeatable starting point.
- The "grill me" skill forces relentless, question-by-question alignment between human and AI to build a shared design concept. This replaces reading long plans; the goal is shared understanding, not an artifact.
- Specs-to-code / vibe coding (iterating on specs without touching code) is a failure. You must keep a handle on the code, shape it, and understand it—code is the battleground.
- The workflow: idea → grilling session → PRD (destination document, not read by the human) → Kanban board of vertical-slice issues (tracer bullets) → AFK implementation by agents → manual QA and code review.
- Implementation should be AFK (away from keyboard) after planning; planning is human-in-the-loop. The Kanban board's blocking relationships enable parallel agent work.
- TDD (Red-Green-Refactor) is essential for agent output—it prevents cheating on tests and forces quality. Feedback loops (tests, type checks) are the ceiling for AI code quality; improve them to raise that ceiling.
- Codebase architecture matters: deep modules (small interface, lots of internal functionality) are testable and AI-friendly; shallow modules (many small files with tangled dependencies) produce bad agents.
- QA is where human taste and judgment are applied. Automating everything yields slop.
- Old software engineering books (Pragmatic Programmer, Refactoring, Philosophy of Software Design) are gold for prompting AI because they verbalize best practices in English.

**Pithy and provocative quotes**

- "Every time you add a token to an LLM, it's kind of like you're adding a team to a football league… It scales quadratically."
- "LLMs are kind of like the guy from Memento, right? They just continually forget, they could just keep resetting back to the base state."
- "I much prefer my AI to behave like the guy from Memento, because this state is always the same, always the same."
- "The specs to code movement… I tried this, I really tried it, and it sucks. It doesn't work because you need to keep a handle on the code."
- "I didn't need an asset, I didn't need a plan. I needed to be on the same wavelength as the AI, as my agent."
- "Bad code bases make bad agents. If you have a garbage code base, you're going to get garbage out of the agent."
- "The quality of your feedback loops influences how good your AI can code. Essentially, that is the ceiling."
- "If you try to automate the sort of creation of the idea, automate the QA, automate the research, automate the prototype, you end up with apps that I feel just lack taste and are bad."
- "We are not producing slop here. We're trying to produce high quality stuff."
- "I don't look at these [PRDs]. The reason I don't look at these is because what am I testing at this point? … I know that LLMs are great at summarization… I have reached the same wavelength as the LLM."
- "TDD is so, so good for places where you can pull it off. And in fact, it's so good that I sort of warp my whole technique around getting TDD to work better."

**Tools, practices, and methodologies**

- **Grill Me skill** — A tiny prompt that makes the AI interview the user relentlessly, one question at a time, about every aspect of a plan until a shared design concept is reached. Used as the first step for any new feature to achieve alignment.
- **Write a PRD skill** — Takes the aligned conversation and produces a product requirements document with problem statement, solution, user stories, implementation decisions, testing decisions, and out-of-scope items. The speaker does not read it; it serves as a destination artifact for the agent.
- **PRD to Issues skill (Kanban board)** — Breaks the PRD into independently grabbable issues using vertical slices (tracer bullets). Creates a directed acyclic graph of tasks with blocking relationships so multiple agents can work in parallel.
- **Ralph loop (AFK agent)** — A prompt and script that runs Claude Code in a loop: picks the next AFK task from the backlog, implements with TDD, runs feedback loops (tests, type checks), and commits. The `ralph_once.sh` script runs a single pass; the full loop is in the speaker's Sandcastle tool.
- **TDD (Red-Green-Refactor) skill** — Teaches the AI to write a failing test first, then the minimal implementation to pass, then refactor. Prevents the AI from writing tests after the fact that cheat, and ensures meaningful test coverage.
- **Improve Code Base Architecture skill** — Scans the repo to identify shallow modules and suggests how to deepen them (create modules with small interfaces and lots of internal functionality) to improve testability and AI navigability.
- **Sandcastle** — A TypeScript library for running AFK agent loops in Docker sandboxes with parallelization. It includes a planner that selects independent issues, creates sandboxes per issue, runs implementers, then a reviewer, then a merger agent that resolves integration conflicts.
- **Token usage monitoring** — A status line in Claude Code that shows the exact number of tokens used, essential for staying in the smart zone.
- **Sub-agents** — Delegating exploration or other tasks to isolated LLM calls that return summaries, keeping the parent context small.
- **Vertical slices (tracer bullets)** — Breaking work into thin, end-to-end slices that cross all layers (database, API, frontend) so that integrated feedback is available immediately, rather than building layer by layer horizontally.
- **Deep modules** — Modules with a small, simple interface and a lot of functionality inside. They are easy to test (wrap a test boundary around the whole module) and easy for AI to reason about. The speaker designs interfaces but delegates implementation to agents.

**Unanswered questions and omissions**

- How to handle code review at scale when agents produce large volumes of code? The speaker admits he doesn't have an answer, only that we'll likely do more code review.
- How does this workflow adapt to messy, real-world team environments with multiple parallel ideas, changing requirements, and domain experts who aren't the developer? Only briefly touched on.
- How to enforce coding standards, architecture constraints, and security policies effectively? The push vs. pull distinction for instructions was mentioned but not a full solution.
- The role of PMs and non-developers in this process—specifically, whether they should "vibe code" tasks—was explicitly dodged.
- Frontend development where visual feedback is critical and AI lacks eyes; only prototyping was suggested as a workaround, not a robust method.
- Long-term maintainability: documentation rot, whether to keep or discard PRDs and plans, and how to prevent stale artifacts from misleading future agents.
- The "dumb zone" boundary of ~100k tokens is presented as a rule of thumb without data or rigorous testing; the claim that it's always "about this" regardless of context window size is unsubstantiated.
- The assertion that LLMs are great at summarization so PRDs don't need human review ignores risks of hallucination or misalignment in the summary.
- Cost management, token economics, and model selection strategy (beyond mentioning Sonnet for implementation, Opus for review) are not discussed.
- Scaling the parallel agent approach to large codebases with many contributors, merge conflicts, and integration testing is glossed over.
- What happens when the user doesn't know the domain well enough to answer the grill me questions? How to bootstrap domain knowledge is not addressed.
- The talk assumes a solo developer or small team; organizational adoption, governance, and stakeholder buy-in are absent.
