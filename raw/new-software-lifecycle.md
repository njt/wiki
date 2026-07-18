---
url: https://www.oreilly.com/radar/the-new-software-lifecycle/
title: The New Software Lifecycle
author: Addy Osmani
date_fetched: 2026-07-18
date_published: 2026-07-15
---

# The New Software Lifecycle

**Author:** Addy Osmani
**Published:** July 15, 2026 • 11 minute read
**Originally appeared on:** Addy Osmani's blog; republished on O'Reilly Radar with permission.
**Draws from:** A Google whitepaper co-authored with Shubham Saboo and Sokratis Kartakis titled "The New SDLC With Vibe Coding."

---

## An agent is a model plus a harness

An intelligent agent consists of a model (one input) and a harness (everything else). The harness includes instructions, rule files, tools, MCP servers, sandboxes, orchestration logic, hooks for deterministic code, and observability systems. The split is roughly 10% model, 90% harness — a figure that "sounds high until you've spent a week debugging one."

Two concrete examples: a team on Terminal Bench 2.0 moved a coding agent from outside the top 30 into the top 5 by altering only the harness. Separately, LangChain added 13.7 points on the same benchmark by changing "just the system prompt, tools, and middleware around a fixed model." Neither touched the model itself.

"Most agent failures are configuration failures." This is encouraging because configuration is fixable without waiting for better models.

## Context engineering is the part that decides your bill

If the harness is the system, context engineering is its most critical control. The paper categorizes agent context into six types: instructions, knowledge, memory, examples, tools, and guardrails. The key financial decision is what goes into static versus dynamic context.

**Static context** loads every turn — system instructions, rule files (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md`), global memory, core guardrails. Reliable but costly, since you pay on every call.

**Dynamic context** loads on demand — skills triggered by task matching, tool results, or RAG-pulled documents. You pay only for what a task actually uses.

The boundary between them should be treated as "a real architectural decision: reviewed in a pull request, versioned like code." The scaling trick is "agent skills with progressive disclosure" — the agent sees minimal metadata at startup, loads full instructions when a task matches, and pulls heavy reference material only when needed.

## Verification is the line between vibe coding and engineering

The spectrum from "vibe coding" to agentic engineering is defined by verification practices. Two mechanisms exist:

- **Tests** cover deterministic parts (this input, that output)
- **Evals** cover non-deterministic parts, split into **output evaluation** (is the final result correct?) and **trajectory evaluation** (was the path — tool calls, reasoning — sound?)

Both are necessary. "An answer that looks right but skipped its checks is more dangerous than one that's obviously broken."

Key leadership advice: "Set the bar at the eval, not the demo." A demo shows an agent *can* work once; an eval suite shows it *reliably* works.

## How each phase actually changes

AI compresses the lifecycle unevenly, and that unevenness is the critical insight. Implementation drops from weeks to hours. Requirements, architecture, and verification remain slow because they involve judgment. Specification quality becomes the new bottleneck, and verification moves to the center of the process.

**Phase-by-phase breakdown:**

- **Requirements** transform from hand-off documents into conversations producing both a spec and a working prototype simultaneously. The agent drafts user stories from a brief and surfaces edge cases.

- **Architecture** "is the most stubbornly human phase." Trade-offs like consistency versus availability depend on business context models can't fully grasp. The developer's role becomes making and documenting structural decisions the agent then implements.

- **Implementation** yields productivity gains of 25% to 39% per surveys, but a METR study found experienced developers going 19% slower on some tasks when counting check-and-fix time. The honest assessment: "AI turns implementation from writing into reviewing."

- **Testing and QA** reverses: tests and evals become the primary way to communicate what "correct" means, wired into a loop — run against benchmarks, cluster failures, fix prompts or tools, check against regression suites, and monitor production.

- **Maintenance** is the most underrated phase. Code previously "too risky to touch" because only its authors understood it can now be read, refactored, and modernized by an agent. Tedious migrations and deprecation cleanups that were perpetually deferred start happening.

The ceiling on all this remains the "80% problem": agents get the first 80% of a feature quickly, but the final 20% — edge cases and cross-system seams — still requires context models typically don't have.

## The economics: Context and routing are financial levers

For leaders, the key metric isn't velocity but total cost of ownership. AI-era economics invert conventional intuitions about what's cheap.

Vibe coding is cheap upfront (a subscription and some prompts) but expensive to run — token burn from unstructured inputs, maintenance taxes from ad hoc code, and security cleanup from fast-generated vulnerabilities. Agentic engineering flips this: more upfront investment (schemas, tests, structured context), less per feature afterward.

The crossover point where "vibe coding costs 3x to 10x more per feature" is illustrative, not a measured constant. The real insight: context engineering and model routing are financial levers. Route hard reasoning to large models and routine work (test generation, code review, CI checks) to small, cheap ones. Quality holds and costs drop.

## The prototype is becoming the production agent

The same terminal workflow producing throwaway scripts can now yield production agents — often by conversing with the coding agent already in use. Building, evaluating, and deploying a real agent (with persistent memory, scoped permissions, eval coverage, and observability) used to require a separate stack. Now it folds into the existing loop.

Google's Agents CLI exemplifies this: after one-time setup, the coding agent picks up lifecycle skills driven in plain language. Coordination between agents runs on MCP (for tools) and A2A (for handing work to other agents).

An Anthropic experiment: a group of agents built a working C compiler in Rust over two weeks, with humans "setting direction and reviewing rather than writing the code."

Two daily modes: the **conductor** (real-time, in-IDE, keystroke-by-keystroke, good for exploration) and the **orchestrator** (async goal-handoff to agents, good for well-specified work like migrations). The shift from conductor to orchestrator is "a skills shift before it's a tooling one."

## The figure for everyone else

Adoption numbers as of early 2026: 85% of professional developers use AI coding agents regularly, 51% use them daily, and roughly 41% of new code is AI-generated.

## Where to start

"AI amplifies whatever engineering culture it lands in, the good parts and the bad parts both." Generation is largely solved; the remaining work is specification and verification — and the systems holding them together. That's the capability worth developing.

The full paper is on Kaggle. Osmani's O'Reilly book *Beyond Vibe Coding* explores specs, harnesses, evals, context, and production-grade engineering in depth.
