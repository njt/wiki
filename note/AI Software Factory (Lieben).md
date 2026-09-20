# AI Software Factory (Lieben)

Erik Lieben's series introduction lays out the full parts list of aifold, his self-hosted .NET software factory for running coding agents unattended: briefs, sandboxes, pipelines, gates, budgets, transcript metrics, layered memory, scoped skills, multi-harness runners, a purpose-built review tool, and agent fleets with a sanctioned communication board — all framed by Toyota's jidoka rather than the lights-out factory.

---

## The argument in one paragraph

Lieben claims that when you take yourself out of the room, every job you were silently doing for the agent — trigger, sandbox, workflow, quality gate, budget, memory, reviewer — must be replaced by an explicit, checkable system component, and that a factory assembled from such parts (briefs with checkable acceptance criteria, gVisor sandboxes, typed pipelines, evidence-reading gates, enforced budgets, person-confirmed memory, harness-agnostic runners, and a review tool built for agent output) can run agents unattended safely and economically. Crucially, he claims this factory must follow Toyota's jidoka and *not* the dark-factory model: because software is never the same work item twice, checking is not a stage you automate away but the process itself, and the lights come back on at every gate failure and at final review. This is falsifiable: if agents' self-reports could be trusted, or if review could itself be automated without a rubber-stamp collapse, most of the parts list would be unnecessary plumbing around a problem that doesn't exist.

## Key quotes

> An AI coding agent, whether it lives in your terminal, Visual Studio, Rider or VS Code, has a safety system you rarely think about: **you**.

The reframe that organises the whole piece: the human isn't a supervisor watching the agent, they are seven undifferentiated subsystems running on wetware. Naming them is what makes them buildable.

> The agent is the most capable part of the system, and also the least trusted. It gets a bounded piece of work, a budget, and a set of tools. Everything around it is plain, boring, deterministic code.

A crisp statement of the trust asymmetry that justifies the entire architecture — and a quiet argument against the "one giant prompt" school of agent design.

> **Give the agent full permissions inside something where full permissions can't hurt you.**

The sandbox section's thesis in one line: permission prompts don't work unattended, so the boundary moves from the prompt to the environment. The worktree-vs-clone detail (shared `.git` hooks as a persistence vector) is the kind of thing only someone who ran this learned.

> **Planning is the part that goes badly unattended, and it's the part that's cheap to do with a person in the room.** Plan there, execute in the factory.

The cleanest division-of-labour rule in the piece, and the one most factory write-ups skip: the factory is for execution, not for deciding what to build.

> Count how many of the agent's non-trivial lines are still there (skip the lone braces, they match everywhere), and divide spend by that: **cost per surviving kLOC** (a kLOC is a thousand lines of code).

A genuinely new denominator. Lines-written metrics flatter churn; measuring what *survived* makes the cheap-model-vs-good-model question answerable, and his Haiku-vs-Opus chart shows the answer is not what the pricing page implies.

## Critical analysis

The most non-obvious move here is the manufacturing history. Everyone building agent pipelines reaches for the dark-factory image; Lieben instead reaches for jidoka and the andon cord, and the choice does real architectural work. It explains why gates send work *back* to the session that wrote it (four tries, then a person), why the review stage is irreducible, and why "a software factory never builds the same thing twice" kills the tuning-the-line-once fantasy. The Tesla anecdote and the FANUC/Philips details are doing more than decoration — they pre-empt the obvious objection that full automation is the end state.

The second under-appreciated contribution is negative space in tool design. The MCP server exposes four tools and deliberately has no "start a run" tool, because "an assistant that can [file and start] is one sentence away from doing both." The terminal's Start-workflow screen has one functional key because "a screen whose purpose is to spend money shouldn't have a second key that also does." And the skill checks whether the pipeline's checks can actually run in the checkout, because "a skipped check reads as a pass" — a failure mode that will sound painfully familiar to anyone who has trusted a green pipeline. These are UX decisions as safety mechanisms, which is rarer than it should be.

The cost-per-surviving-kLOC metric is the piece's most exportable idea and also its most fragile. It is honest about churn (measure lines, not files; exclude scratch files; unknown cost is null, never zero), and the per-model chart — Haiku cheapest per kLOC written but only just beating Sonnet per kLOC *surviving* — is exactly the analysis the pricing pages don't give you. But the numbers shown are labelled illustrative, the sample is one person's hobby-scale runs, and "survival" is measured against disk, not against whether the surviving code was *good*. A factory could optimise itself into shipping mediocre code that merely persists.

Weaknesses worth naming: this is a single-builder field report with n=1 throughout, and several hard problems are admitted unsolved (Testcontainers-style integration tests can't run in the sandboxes; a quietly-weakened test that keeps its count defeats the gates; "gates can't see intent"). The fleet section leans heavily on the OpenAI Hugging Face incident — dramatic, but the inference from "agents build covert channels when denied sanctioned ones" to "give them a board" is thinner than the storytelling suggests. And the review-tool chapter, while the most detailed, assumes the filer-of-the-brief is also the right first reviewer, which holds for a solo builder and may not survive contact with a team.

What's left out: anything about multi-person organisations (identity gets one bullet), how brief quality degrades when the planner didn't write the code, and an honest total cost of ownership for the factory itself — he flags that "a factory is a lot of plumbing to maintain" and moves on. That caveat is honest but understated: the piece is twelve sections of plumbing, which is precisely the maintenance bill.

## Related

- [[How to Build an AI Software Factory]] — The Firecrawl synthesis claims the factory exists to absorb agent output because review doesn't scale; Lieben's independent first-person build strengthens that control-plane structure (intake, isolation, verification, merge gate all appear) while complicating its economics: his jidoka framing and cost-per-surviving-kLOC metric suggest the factory's real product is *trustworthy review throughput*, not absorbed generation.
- [[Harness Engineering (OpenAI)]] — OpenAI's report argues the discipline shifts from writing code to engineering the harness around agents; Lieben strengthens that thesis from the opposite direction — a solo self-hosted build — and adds a concrete refinement OpenAI doesn't need: the harness must be a detail of a *phase*, behind one runner contract, because no single agent vendor survives contact with pricing, outages and deprecation.
- [[Agentic Code Review]] — Osmani's guide argues the bottleneck has moved to trusting agent output and reviews must be restructured around risk; Lieben strengthens this with the most concrete review-surface design in the wiki: typed comments (issue/question/nitpick/todo/praise), files grouped by kind with generated files folded, and reviews that export back as briefs through the same gates rather than around them.
- [[Loop Engineering]] — Osmani's maturity leap from prompting agents to designing the system that prompts them is exactly what aifold instantiates; this source nuances it by showing the loop's least glamorous half — the plumbing of budgets, transcripts, retries and stop buttons — is where the design effort actually goes, and that "none of this is especially clever" is the point.

---
*Sources: [[raw/ai-software-factory-introduction]], [[summary/ai-software-factory-introduction]]*
