# How Always-On Agents Work (Stanford CS146S)

Lee Robinson's CS146S lecture is a rare end-to-end walkthrough of a production always-on agent ("bot") architecture — the server/VM split, durable workflows, subagents, prompt caching, and layered memory — delivered as teaching material, with candid notes from building it at SpaceX.

---

Lee Robinson (ex-Cursor/Vercel, now agent infrastructure at SpaceX) surveys four eras of developer-AI interaction — copy-paste from ChatGPT, terminal harnesses, dedicated apps with parallel agents, and now always-on bots — and argues the new era's defining properties are: the agent has its own computer, it has memory, it learns skills on the job, and it keeps working after you close the laptop. Crucially, it's "more like a colleague, where it knows when to be quiet."

The architecture rests on a few load-bearing decisions. The client is deliberately thin: exactly one tool, `send_to_user`, connects client and server, so any surface (mobile, Slack, email, desktop) speaks the same protocol while rendering richly. The agent's computer is a Firecracker VM that boots on demand, snapshots, and sleeps — because "there's not enough computers in the world" to run them always-on. The agent loop lives on the server, never in the VM, so the VM can be rebuilt (say, for a zero-day patch) without touching the agent or losing state. Long-running work runs as durable workflows on Temporal: on failure it "picks right up on step three" instead of replaying steps one and two.

The context-engineering section is the most transferable material. Prompt caching dominates cost, so "a cache miss is kind of like a bug in the system"; the team compacts while the cache is still warm, hashes prompt sections to keep ordering stable across deploys, and even prefetches/warms the cache when a user opens the chat and starts typing. Subagents give the illusion of infinite context by keeping expensive tool output out of the main conversation. Tool discovery is dynamic: only tool names live in context, schemas are read on demand from files — "it's files and a file system all the way down."

---

## Key quotes

> "It's more like a colleague, where it doesn't always immediately send you a message. It knows when to be quiet."

The framing device for the whole talk. The hard part isn't making agents run forever — it's the "do I need to bug the user about this or not?" heuristic, which he admits is "not perfect yet" and names as an open problem (proactive vs annoying).

> "Funnily enough, that's actually more secure than the fatigue of having to watch hundreds of shell commands and approve each one... if you did a blind test, most of the time the model's actually doing a better job."

On "auto review" — a separate model reviewing every command a bot runs. This is a strong claim for [[AI Code Review]]-style thinking applied to operations: human approval fatigue is itself a vulnerability, and model review beats rubber-stamping humans.

> "A cache miss is kind of like a bug in the system, really."

The sharpest single line in the lecture. Once agents run continuously and tokens flow at scale, cache discipline stops being an optimization detail and becomes a correctness/expense concern — whole incidents (the OpenClaw-era stampede on subscription plans) are retold as cache-miss epidemics.

> "You have to internalize and assume that the models today are the worst they'll ever be. Which means that in six months you might need to completely rebuild the UI."

The "delete the product" principle carried over from Cursor to SpaceX. Build as little as possible for the next generation of model, not the current one — a deliberate counterpoint to platform maximalism.

> "Every new product is going to have to have some kind of agentic side to it, either integrating, or it will become a headless piece of data for another agent to use."

His closing framing (attributed to an OpenRouter take on X): the sidebar-chatbox app is the new CRUD app — the 2005 stack that everyone built and nobody argued about.

---

## Themes

- #pattern — thin client, thick server: one tool (`send_to_user`) on the client boundary, everything heavy server-side
- #concept — durable workflows over job queues; state persisted so VMs are disposable and work resumes mid-task
- #concept — context engineering as the core harness discipline: caching, compaction-while-warm, subagent delegation, dynamic tool discovery
- #concept — layered memory: persistent facts → dated logs → ephemeral scratchpads → everything else greppable in files

## Analysis

This is the most concrete public description of a production always-on agent stack I've seen in lecture form, and its credibility comes from specifics rather than vision: Firecracker snapshotting, Temporal, cache-warming on keystroke, accessibility-tree scraping before pixel-clicking. The through-line is an economic one — nearly every design decision is justified by tokens or dollars (context budget, cache locality, delegation as cost control), which is the honest way to reason about agents that run for weeks.

The weak spot is the same one Robinson flags himself: the social heuristics (when to interrupt, how to behave in group chats) are hand-waved as "heuristics" while the infrastructure gets precision. The Moltbook / Hugging Face incident references suggest the industry still doesn't know how to train for multi-agent etiquette — a gap between the solved engineering and the unsolved behavioral layer.

Also worth noting: the talk quietly assumes an org that can afford a dedicated VM per agent and a Temporal fleet. For individuals and small teams, the pattern survives (durable state, files-as-memory, quiet-by-default), but the infrastructure collapses to something much smaller.

## Relations

- This lecture is essentially the systems-architecture companion to [[How Hightouch Built Their Long-Running Agent Harness]] — Hightouch covers similar long-running-harness ground from one practitioner's codebase, while Robinson zooms out to the general building blocks (durable workflows, VM lifecycle, auto review) and names the vendors (Temporal, Firecracker) behind them.
- It strengthens [[Scaling Long-Running Agents]] with the cost side of the story: compaction scheduled against warm-cache windows, prompt section hashing across deploys, and cache-warming on user intent are concrete tactics for the same problem that page describes more abstractly.
- It nuances [[How AI Agent Memory Works]] with a sharper taxonomy — per-prompt user facts, dated logs, ephemeral scratchpads, greppable files — plus the "garbage collection of facts" framing, and adds the training dimension (models learning what to keep) that pure architecture treatments skip.
- It complicates [[Headlong — A Microharness for Persistent Agents]]: Headlong argues persistence can be micro and minimal, while Robinson's stack is deliberately heavy (dedicated VMs, Temporal, auto review) — the tension between "just enough harness" and "colleague-grade infrastructure" is unresolved and probably task-dependent.

---
*Sources: [[raw/stanford-cs146s-how-always-on-agents-work]], [[summary/stanford-cs146s-how-always-on-agents-work]]*
*Last updated: 2026-11-02*
