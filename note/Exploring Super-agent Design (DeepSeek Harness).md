# Exploring Super-agent Design (DeepSeek Harness)

DeepSeek Harness team's architecture essay argues that the unit of agent work should stop being the isolated, human-driven session and become one continuous Super-agent — Continuous, Productive, Proactive, Persistent — assembled from five classic systems-engineering modules: a Router/Worker two-level architecture, event-driven Triggers, full-duplex multi-channel interaction, compute–state–environment separation, and an external Watchdog. Its central diagnosis: single-task agent quality is now model-limited, but multi-task throughput is limited by human attention, and no ReAct loop can fix that — "they have to come from the system architecture around it."

---

## Key Quotes

> "a person can usually follow only 3 to 5 active sessions at once... a single task is limited by model capability, but multi-tasking is limited by human attention."

The framing that motivates everything else. This is the same attention-bottleneck argument seen across the wiki — review bottlenecks, supervision fatigue — but stated as a capacity constraint on sessions rather than on code review, and it justifies hiding sessions from users entirely.

> "Sessions should be an implementation detail of the system, not a mental burden on its users."

The boldest product claim. A session currently does two jobs — context management and project management — and decoupling them means chat windows stop mapping to underlying sessions. The open-problems section honestly admits nobody knows what interaction replaces them ("the chat window became the center of interaction partly through path dependence").

> "The Router layer is designed not to execute tasks itself. It only interacts with humans and dispatches tasks, and it can run without a Bash tool at all."

A structural, not prompt-level, fix for the delegation problem. Their ablation shows why it matters: with a neutral "use a subagent when appropriate" prompt, the single-session baseline's model delegated in **zero of five runs**; a delegation-first prompt got 4/5 with a long tail. Two-level architecture takes that judgment out of the model's hands — a systems answer to a prompting failure.

> "Wasting the context window is wasting the model's intelligence."

Inline & Spill treats the context window as a cache with locality, borrowing from compilers and OS memory management: inline what's needed soon, spill the rest behind a pointer, and let Chain of Recall walk a reference graph hop by hop. A crisp vocabulary for event-injection design that most harnesses do ad hoc.

> "Many anomalies are not crashes: the process is still running, but the model is stuck in an unproductive loop or has drifted from its goal."

The Watchdog's premise: liveness probes can't see semantic failure, so a long-running agent system needs an independent Control Loop that judges *progress*, not just process health — and is itself an agent session maintainers can interrogate in natural language.

> "If a server-side intrusion or a prompt injection compromises the Agent Loop, every endpoint device connected to it could be taken over... the agent needs a way to use credentials without ever seeing them."

The most candid part of the paper. BYOM means the cloud brain reaches into machines holding uncommitted changes and private credentials, and the authors admit the whole architecture "implicitly assumes that the cloud and the endpoints trust each other." They flag the problem and leave it open.

## Key Themes

#concept #pattern #agent-architecture #event-driven #systems-engineering

## Analysis

The essay's real thesis is methodological, not architectural: **classic computer systems ideas — Fast/Slow Path, caching and locality, speculative execution, compute-storage separation, external watchdogs — transfer almost wholesale to agent systems.** That framing is why the piece reads better than most agent-design writing; it's disciplined borrowing rather than novelty inflation, and the authors repeatedly note that implementations will change while the design ideas last longer.

It is also honest about its evidence. The experiments are explicitly "indicative only": synthetic workloads where coding is "simulated by a tool that simply consumes time" — so Section 7 measures scheduling and routing, not code quality. The strongest finding (structural delegation beats prompt-level delegation) is real but measured on mock Workers; the batching result (amortized decision cost falling 6.4s → 1.6s as bursts rise) is the cleanest data in the piece. The token-cost admission cuts the right way too: Super-agent spent 23.2M cache-read tokens per run vs 6.1M for the single session — trading tokens for stable completion times, which is a real product decision the authors surface rather than hide.

What's missing is any treatment of the harder half of the loop: verification. The proactive examples ("silently reproduce and fix," speculative PRs opened while a discussion is still ongoing) assume the agent can tell when its fix is right, and the paper says nothing about how. An agent that autonomously opens PRs without being mentioned is exactly the failure surface [[Agentic Code Review]] documents — the Wiki's review-automation sources describe the gate this architecture quietly presumes. Likewise, the Router layer is a single point of failure for *correctness*, not just throughput: it owns all global memory and all user-facing replies, and the paper's answer to overload (sharding) sacrifices exactly the continuity it was built for — "this trade-off will remain until the global memory problem is solved," as candid a statement of an unsolved problem as you'll find.

Positioning against neighbors: this is a system-level, product-shaped version of the same always-online turn that [[How Always-On Agents Work (Stanford CS146S)]] teaches and that [[Headlong — Persistent Agent Microharness]] pursues from the opposite direction (Headlong dissolves the session into one thought stream; DeepSeek keeps sessions but makes them invisible plumbing). And it confirms, with a shipped internal instance and ablations, the async-transport argument of [[All Your Agents Are Going Async]].

---
*Sources: [[raw/2116857257245987236]], [[summary/2116857257245987236]]*
*Last updated: 2026-11-02*
