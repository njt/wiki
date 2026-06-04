# Your Coding Agent Should Do AI System Engineering

Ben Burtenshaw (Hugging Face) argues that coding agents are ready for the hardest AI engineering: writing CUDA kernels, fine-tuning models in a single prompt, and running multi-agent research labs. The key dependency isn't model capability — it's standard, open repositories as the substrate agents operate on.

---

## Key Quotes

> "Your coding agent should do AI systems engineering."

Burtenshaw's thesis. Not "could eventually" — "should, now." He means custom kernels, training runs, experiment loops. The stuff that currently requires specialized engineers and weeks of setup.

> "In most cases, memory is usually the bottleneck."

On why CUDA kernels matter: an H100 can do a petaflop per second but only moves 3 TB/s through memory. GPUs sit idle waiting for tensors. Custom kernels like Flash Attention increase arithmetic intensity — more computation per memory transfer. Agent-assisted kernel writing makes this optimization accessible to engineers who don't live in CUDA-land.

> "It takes a task from being zero shot to being few shot."

On skills — file-based context (markdown docs, example scripts, benchmarking harnesses) maintained alongside project repos. An agent with a CUDA kernel skill writes better kernels than the same agent with just the prompt. This is the practical implementation of "context quality matters more than model quality" ([[Components of a Coding Agent]]).

> "Even though abstracted APIs are really useful… if we have a layer that we can't get behind, that is a ceiling."

The case for open primitives over polished abstractions. Tracheo's parquet data layer, for example, lets agents read and write metrics directly rather than going through a dashboard API. The agent doesn't hit a wall where the abstraction stops; it can see and manipulate the raw data. Compare with [[Smart Models Dumb Pipes]] — the end-to-end principle applied to agent infrastructure.

> "We keep the GPUs warm."

A throwaway line that captures the economic argument: idle GPUs are the real cost. Agent-driven experiment scheduling maximizes utilization by keeping the compute pipeline saturated.

## Key Themes

#coding-agent #multi-agent #cuda #fine-tuning #research-automation #open-primitives #skills #huggingface

## Critical Analysis

**The skills insight is the most transferable takeaway.** File-based context (skills) turning zero-shot into few-shot is something every coding agent user can implement today — drop a markdown file with examples next to your project, and your agent gets better. This generalizes Burtenshaw's kernel-writing example to any domain. It's the same pattern behind [[CLAUDE.md]] and [[MinMax Skills]], but Burtenshaw frames it as a measured performance boost rather than a workflow preference.

**The three "bosses" are a useful autonomy ladder.** Interactive kernel writing → zero-shot fine-tuning → auto-research lab. Each step removes a human from the loop. The jump from Boss 2 to Boss 3 is the interesting one: multi-agent coordination with role specialization (researcher/planner/worker/reporter) is the same pattern that shows up in [[Agent Orchestration]] across every team that's tried it — Cursor, Hightouch, Maestro. When multiple teams converge on the same architecture independently, it's not a coincidence.

**The unanswered questions are the real story.** Burtenshaw shows agents generating CUDA kernels with 94% speedup, but there's no mention of correctness verification beyond benchmarking. A fast wrong kernel is worse than a slow correct one. Same for the auto-research lab: no cost controls, no experiment pruning, no failure detection. These are the problems that make [[Designing Agentic Loops]] and [[How Hightouch Built Their Long-Running Agent Harness]] hard — the happy path is easy; the failure modes are where engineering happens.

**Open primitives as moat-building.** Hugging Face's pitch — "standard repos on the Hub" — is infrastructure marketing dressed as engineering philosophy. But the argument is correct even if the motivation is mixed: agents absolutely work better with direct access to data layers than with API wrappers. The question is whether the Hub becomes the standard or one of many.

**Comparison with existing work.** The auto-research lab is a specialized instance of the planner/worker/judge pattern that dominates [[Agent Orchestration]]. The researcher agent using HF Papers CLI as a literature scout is novel — most coding agent orchestration doesn't include literature review as a first-class step. The skills approach is philosophically aligned with [[Honey I Shrunk the Coding Agent]]'s finding that the scaffold matters more than the model, but Burtenshaw's implementation (file-based context) is simpler and more portable than Inbar's custom harness redesign.

## Related

- [[Agent Orchestration]] — the planner/worker/judge pattern that the auto-research lab instantiates
- [[Components of a Coding Agent]] — Raschka's taxonomy; the harness matters more than the model
- [[Honey I Shrunk the Coding Agent]] — same insight from the other direction: vary the scaffold, hold the model constant
- [[MinMax Skills]] — parallel skills ecosystem for coding agents
- [[Designing Agentic Loops]] — the failure modes Burtenshaw doesn't address
- [[How Hightouch Built Their Long-Running Agent Harness]] — real-world long-running agent challenges
- [[Smart Models Dumb Pipes]] — the end-to-end principle Burtenshaw's open primitives argument echoes
- [[Scaling Long-Running Agents]] — multi-agent at scale; what the auto-research lab looks like when it runs for weeks

---
*Sources: [[raw/20260604-your-coding-agent-should-do-ai-system-engineering-ben-burtenshaw-hugging-face]]*
*Original talk: https://www.youtube.com/watch?v=JomVvNDjGb8*
*Last updated: 2026-06-04*
