---
url: https://randomlabs.ai/blog/slate
title: "Slate: moving beyond ReAct and RLM"
author: Random Labs
date_fetched: 2026-05-14
date_published: unknown (estimated mid-2025 based on references to "early 2025" as past)
topics:
  - agent-architecture
---

# Slate: moving beyond ReAct and RLM

Technical report from Random Labs introducing a new agent architecture pattern, demonstrating how single-threaded agents can generalize beyond ReAct and RLM.

Note: This source was extracted from a client-side-rendered React SPA (randomlabs.ai). The full text was recovered from the JavaScript bundle. Some formatting and reference numbering may be imperfect. All substantive content is preserved.

## Introduction

In this technical report, we introduce a new agent architecture pattern, and demonstrate how single-threaded agents can generalize beyond ReAct and RLM.

Our goal at Random Labs is to build generalized, non-benchmaxxed, end-to-end agents for software engineering. The contents of this report bring us one step closer to this goal.

We begin by examining a series of problems faced by modern LLM based agents: long horizon tasks, strategy vs tactics, and working context management. We explore existing solutions to these problems and their limitations. After enumerating the problems faced by modern agents, we describe Slate's architecture: a thread-based episodic memory system that solves all of them simultaneously.

Building agents that generalize requires solving three compounding problems: long-horizon task execution, the balance between strategic and tactical reasoning, and working memory management. Each of these is tractable in isolation — the difficulty is that they interact.

## Understanding long horizon tasks

Long-horizon tasks are path-dependent (tasks where the required actions depend on each other), and the minimum number of steps to succeed exceeds what a minimal harness can do. A minimal harness is defined here as a limited tool-calling loop around a model with no additional planning or memory infrastructure. Terminus and Simple Codex both fall into this category while most agents you actually use to get work done do not. [4][5]

In order to solve this type of task, the agent requires three things: adequate working memory so the model can attend to the right context at the right time, a balance of strategic and tactical execution so the model both plans well and executes correctly, and the ability to integrate new information discovered throughout the task without losing track of the overall goal.

Models cannot attend uniformly across their context window. The model's ability to attend to the information degrades as the context length grows. The usable portion, up to but not including the degraded region, is working memory. Dex Horthy coined the term "Dumb Zone" for the part of the context window where retrieval quality drops. [6] Working memory is effectively the context window up to that point. [25]

[Figure: Context Window & Dumb Zone — Context windows are not uniformly usable. The right edge degrades in attention quality as the window fills up.]

[Figure: Performance Degradation vs. Input Length — via Hong, Troynikov & Huber (Chroma research). All four frontier models degrade non-uniformly as input length grows, even on simple tasks.] [24]

## Strategy and Tactics

Strategy is open-ended planning based on knowledge that guides the system towards the goal, and tactics are learned, local action sequences which help materially take steps towards the goal. It's serendipitous that this distinction maps directly onto how RL has historically solved games like go and chess. In chess, traditional engines like Stockfish brute force through a move tree (a sort of tactic based search algorithm) [16]. In contrast, self-play RL has yielded systems that learn which positions matter strategically.

> "In several games AlphaZero sacrificed pieces for long-term strategic advantage, suggesting that it has a more fluid, context-dependent positional evaluation than the rule-based evaluations used by previous chess programs." [11]

[Figure: Brute Force vs. Value / Policy Execution — Brute force evaluates every node at every depth. Value/policy guided search uses the value network to judge positions strategically and the policy network to select moves tactically — exploring far fewer nodes to reach stronger play.]

Go makes the separation even more explicit: strategy covers influence, territory balance, and whole-board thinking, while tactics cover reading (calculating specific local sequences) and life-and-death problems. When DeepMind built AlphaGo, this split was literally architected in: the value network handles positional judgment (strategy) while the policy network handles move selection (tactics). [14] Research probing AlphaZero's internal representations during training found that tactical concepts — material value — are learned first, followed by strategic concepts like king safety and mobility. They emerge separately, at different training stages, in different layers of the network. [12]

[Figure: AlphaZero: Concept Emergence by Training Stage — Tactical concepts (material, space) are learned within the first 32k steps. Strategic positional concepts (king safety, threats, mobility) emerge at 32k+. Long-horizon trade-off reasoning develops last. McGrath et al., PNAS 2022.]

"...piece value is a keystone concept, developed first. Subsequently, issues around mobility (king safety, attack, and defense) arise. Finally, there is a refinement stage, in which the network learns to make sophisticated trade-offs.." [12]

The paper's qualitative evaluation by former world champion Vladimir Kramnik confirms the order directly. At 16k steps AlphaZero loses on material; by 32k it has a solid grasp of piece value. The 32k-to-64k leap is dominated by king safety in imbalanced positions. Beyond 128k the gains are in knowing which attacks will succeed — accepting material sacrifices and converting them — rather than in positional or endgame play. Tactical knowledge first, strategic judgment second.

### Software Engineering as an open-ended long horizon game

We see software engineering as a more open ended and infinite game where both strategy and tactics are relevant depending on the task they are applied to. For example, remembering and executing a bash command is a simple tactic. Designing a schema so that it is backwards compatible as it changes is more strategic.

[Figure: Strategy vs. Tactics Spectrum — Software engineering tasks span a spectrum. Tactics are immediate and executable; strategy requires reasoning about future states and tradeoffs.]

This can be seen in the fact that if a model is asked to write a plan or think through a task step by step, it's actually first being given a knowledge retrieval task, and then an agentic execution task. The initial planning/thinking can be viewed as the model retrieving knowledge and strategizing about a path to the solution, and then using tactics to execute the plan.

> A brief aside -- most rules in AGENTS.md files are actually tactical ex. "Never run db commands".

No existing approach solves all of the above problems simultaneously. Each approach accepts a tradeoff of solving one or two at the expense of the others.

## Solving working memory and defeating the Dumb Zone

The first solution that comes to mind is to compress the context for the model! Surely we can reliably drop irrelevant context periodically and solve our problem. This strategy is known as compaction.

Compaction is one naive solution to working memory in isolation.

Compaction is largely unsolved. Most is actually lossy compression (despite having gotten better).

> In early 2025 (around may) we built (one of) the first instance(s) of a sliding window based agent that could run for incredibly long session lengths (up to 2 days reported by our users). This agent is deprecated, but is available as an npm package: `npm i -g @randomlabs/slatecli`

There are a few instances of working but lossy compression in the wild:

- Compaction in claude code (notoriously bad)
- The now infamous Ralph Wiggum loop by Geoffrey Huntley [17]
- Amp handoffs (a crowd favorite, but requiring guidance from the user) [18]

Amp probably has the most interesting implementation here since a handoff is designed to bootstrap a new fresh agent session.

The main issue with compaction is that it is not deterministically lossy, which means we can unpredictably lose important information.

To avoid the lossiness of compaction, we can instead try to isolate the unimportant context. This is where subagents come in.

Subagents are a second naive solution to working memory in isolation. Subagents work relatively well. They isolate context. This isolation means that the naive implementation fails to transfer information across context boundaries since all it returns is a response message (see codex/claude-code subagents).

## Markdown Planning

To make sure we maintain coherence across different parts of a task, compactions, and isolated subagent contexts, we can plan upfront.

Markdown plans are also one method of balancing strategy and tactics. By asking a model to plan the task out, it forces the model to use its knowledge to strategize about the task, which broadly provides a much much better outcome than directly exploiting its own learned behaviors. Giving the model the tactic to track the task progress in the doc allows the model to repeatedly refresh its understanding of its strategy throughout the task and stay aligned.

As models improve and are trained on this style of strategizing through markdown plans, the tasks the model can complete with just a simple markdown file will necessarily increase in scope. However, there will likely always be a difference between planning v.s. directly exploiting the learned behaviors.

> We can describe this as knowledge overhang. The knowledge that a given model has access to theoretically, but can't access tactically without a trick like "think step by step" or by planning in files. [22]

## Threads and Episodes: Slate's Architecture

[Figure: Subagents vs Threads — Subagents each run in their own isolated context and communicate only via message passing. Threads share context explicitly. The orchestrator passes context into each thread, episodes return back, and one thread's episode can become another thread's input.]

### Solving Episodic Memory with Threads

The steps a thread takes while completing an action constitute an episode. This gives us a tractable form of true episodic memory in LLMs.

Episodic memory is the compressed representation of a completed episode: only the important results are retained, not the full tactical trace of every step taken to reach them. Subthreads do not do back-and-forth message passing with the main thread. Instead, they execute, and the episode is returned. This built-in completion boundary is what makes compaction natural in Slate's architecture.

Episodes can also be used as direct inputs to other threads. This makes threads composable. A thread can be initialized with the episode of a prior thread inheriting the useful conclusions and work history without inheriting the full context. This composability is what makes a thread-based architecture maximally expressive as a primitive, and what distinguishes it from naive subagent designs that only pass back a single response string.

The result of thread-based execution is a system that decomposes tasks implicitly and adaptively.

The mechanism: the orchestrator uses threads by reference, giving it semantics for complex context routing.

The result is a system that decomposes tasks implicitly and adaptively. The orchestrator manages planning and decomposition as it goes. It's not forced to commit to a static plan upfront. But it is forced to externalize that decomposition in useful units of work that can be compressed and referenced later. Frequent synchronization means the orchestrator can also update its strategy when new information arrives mid-task.

[Figure: Thread weaving — bounded worker episodes dispatched from and synchronized back into one orchestration thread. Threads T1/T2/T3 run independently; their episodes become inputs to subsequent work.]

### Threads as Processes: An OS view into LLM systems

Threads and episodes map directly onto an OS style framing. [26]

Specifically, Karpathy's LLM OS describes the LLM as an operating system kernel: managing context (RAM), spawning processes (tool calls, subagents), reading and writing to storage (files, memory), and coordinating I/O across peripherals (browsers, terminals, APIs). Just as an OS kernel doesn't execute application logic itself, the main thread LLM schedules tasks, manages resources, and maintains process state in order to route work through the system.

Slate's thread architecture maps onto this directly. The orchestration layer is the kernel. Threads are isolated processes. Episodes are the process return values: compressed summaries of what the process did, committed back into the kernel's working memory. The filesystem, terminal, and web are the peripherals. The model's context window is RAM.

Slate's episode architecture is a direct answer in that framing: instead of letting RAM fill until the process crashes, each thread return is a natural opportunity to decide what gets retained, what gets compressed, and what gets discarded.

### Long horizon task bottlenecks

The real bottleneck in long-horizon agentic tasks is context management, not model intelligence. Models are already capable enough to solve many more tasks than they currently succeed at due to the knowledge overhang. The gap is a systems problem, not a capability problem.

What's remarkable about Slate is that our routing works at all. The models seem to understand how to route context throughout the system in ways that are useful and appropriate, without being explicitly trained to do so. We leave a formal analysis and benchmarking of this routing behavior as future work.

And we leave it as an exercise for the reader to experience.

Today we are releasing this agent into open beta. You can use it by visiting our home page or running `npm i -g @randomlabs/slate`

### Interesting Observations

A few results that surprised us during development and testing:

- **Massively parallel execution in practical workflows.** Real software tasks decompose naturally into parallel thread workstreams. The orchestrator can dispatch several threads simultaneously and synthesize their episodes before continuing. This is qualitatively different from sequential step-by-step agents and in practice it seems to be faster.
- **Cross-model composition.** Using Sonnet and Codex together across the same task works well. The episode boundary acts as a clean handoff between models with no loss of context coherence.

## References (partial)

- [4] https://www.tbench.ai/news/terminus (Terminus)
- [5] https://www.tbench.ai/leaderboard/terminal-bench/2.0/simple_codex/ (Simple Codex)
- [6] Dex Horthy, "Dumb Zone" (context window degradation)
- [11] Silver et al., "Mastering Chess and Shogi by Self-Play" (AlphaZero)
- [12] McGrath et al., PNAS 2022 — https://www.pnas.org/doi/10.1073/pnas.2206625119
- [14] DeepMind, AlphaGo — https://deepmind.google/blog/innovations-of-alphago/
- [16] Stockfish (traditional chess engine)
- [17] Geoffrey Huntley, Ralph Wiggum loop — https://ghuntley.com/ralph/
- [18] Amp handoffs
- [22] Knowledge overhang concept
- [24] Context Rot — https://research.trychroma.com/context-rot (Hong, Troynikov & Huber)
- [25] Working memory / context window
- [26] Karpathy, LLM OS — https://x.com/karpathy/status/1935518272667217925
- RLM (Reasoning Language Model) — https://alexzhang13.github.io/blog/2025/rlm/
- Chain-of-thought prompting — https://arxiv.org/pdf/2201.11903
- Manus context engineering — https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
- Cognition (Devin) on multi-agents — https://cognition.ai/blog/dont-build-multi-agents

## Product Details

- Install: `npm i -g @randomlabs/slate && slate`
- Description: "a generalist software agent built for swarms"
- Tagline: "A brand new architecture for massively parallel swarm orchestration"
- Mission: "to create general software agents and interfaces that allow engineers to maximally leverage them"
- GitHub: https://github.com/entropy-research/slate-cli
- Features: built-in compaction, implicit planning, cross-model composition, privacy modes, long-running sessions
