# Building Autonomous Goal Loops That Deliver

A design for the harness that turns a coding agent from "a loop that moves" into "a loop that delivers" — the unnamed jx0.ca author's follow-up to "The Convergence Problem." The thesis: what sits between the agent's turns and the human's rounds is not a better retry prompt but a machine that exposes a real failure, locates the missing capability, and preserves the lesson across sessions.

---

## Key Quotes

> "The agent keeps moving, but movement is not the problem. The tests only protect what we already know, and the important failures sit outside them. A screen can look complete while it stores nothing."

The article's sharpest diagnosis, and the reason "runs until tests pass" is the wrong loop. A passing test suite can certify behavior you already earned but cannot tell you the next capability to build. This is exactly the gap [[Poor Man's Loop Engineering]] leaves unexamined — it assumes the reviewer compares against a goal that already exists — and it inverts the comfortable assumption of [[Designing Agentic Loops]] that tests are the force multiplier that makes loops converge.

> "It needs a harness that does three things. The harness must expose a real failure, locate the missing capability, and preserve the lesson after the session ends. That is a different machine from an agent with a retry prompt."

The one-sentence definition. A retry prompt generates more attempts; a harness generates *knowledge*. The "preserve the lesson" clause is the part most loop-engineering writing skips — [[Loop Engineering]] lists state as a sixth component but treats it shallowly, whereas here persistence is the load-bearing column.

> "One gap does not mean one file. It means one causal claim. The missing capability may cross the data model, tool contract, runtime, agent instruction, interface, and effect check."

The round closes `request → representation → operation → persistent effect → visible proof` end to end. This is the discipline behind "one round closes one gap" — and the reason "five layers of unfinished possibility" lose to one narrow path that works from request to effect.

> "The scorer is part of the threat model."

Five words that most eval-heavy writing never reaches. An autonomous loop optimizes whatever measure it can reach, including a bad one, so the measure itself needs an authority model. **Free** files are ordinary levers; **Propose** files cross a boundary and need a human; **Frozen** files *define the exam* — corpus, fixture, rungs, score rules, rubric. The loop can add a regression check but never weaken one, and never change a product lever and its measure in the same round (otherwise it grades its own repair). This is [[Guardrails and Feedback Loops]] applied to the scorer rather than the agent.

> "The quality of an agent loop is not how long it runs or how many commits it produces. It is what remains after a round. A good round leaves a working capability, evidence that it works, and a harness that will not pay for the same lesson twice."

The closing, and the metric that separates this from the throughput fetish of "commit count" loop writing. It's a direct rebuke to the longest-run-wins framing that [[Harness Engineering (OpenAI)]]'s "3.5 PRs/engineer/day" statistic can accidentally license.

## Key Themes

#concept #pattern #tool #agentic-loop

- **#concept — The harness as a machine, not a retry prompt**: three functions — expose a real failure, locate the missing capability, preserve the lesson. This is the autonomous counterpart to [[Poor Man's Loop Engineering]]'s explicit "the loop isn't that hard to get started; making it truly autonomous is."
- **#concept — Floor vs. direction**: a deterministic floor (tests, validators, schemas, effect checks) preserves earned behavior; direction comes only from driving a real request and inspecting the shortfall. The floor starts green and regressions become the next job; it cannot say what to build next. Separating them "keeps the loop honest."
- **#concept — Persistent effects over visible results**: "software can describe the correct action without performing it." The scorer must read what *changed*, not what was *rendered*.
- **#pattern — Four roles, two agents kept separate**: development agent (knows the code), driver (approaches through the user surface), scorer (reads what happened), controller (picks the next gap). The product agent gets a fresh conversation and only product-exposed tools, or the test is worthless.
- **#pattern — Free/Propose/Frozen authority model**: the scorer and the measure are themselves constrained, because a loop optimizes whatever it can reach.
- **#tool — `goals/` as the control plane**: `goal.md`, `facts.md`, `plan.md`, `LOOP.md`, `STATE.md`, `corpus/`, `rounds/`. `LOOP.md` changes when the process changes; `STATE.md` after every round — "sessions disposable without making the work forgetful."
- **#concept — Three loops at three speeds**: the product loop (one capability a round), the harness loop ("what cost time that a rule or check could prevent?"), and the human-owned direction loop (continue, redirect, or stop after ~three rounds).

## Critical Analysis

**The genuine contribution is the floor/direction split.** Most loop-engineering writing — [[Loop Engineering]], [[Designing Agentic Loops]], [[Poor Man's Loop Engineering]] — treats tests as *the* mechanism that makes loops work, and stops there. This article is the first I've read to say plainly that tests are a *floor*, not a *compass*: they protect what you already earned and cannot tell you what to build next. That distinction dissolves a lot of false confidence. A green suite is evidence of nothing about the next capability, and a loop that only runs until tests pass will quietly build the wrong thing with a clean bill of health.

**The "effects matter most" clause is the other load-bearing idea.** "A screen can look complete while it stores nothing" is the failure mode of every agent that stops at a rendered UI. Requiring the scorer to read persistent effects, not visible results, is the difference between a demo and delivery — and it's why the article keeps the driver and scorer separate roles even though a single implementation could do both. Independence of measurement from generation, extended one layer deeper than the usual writer/reviewer split.

**The authority model is the part most people will skip, and the part they most need.** Free/Propose/Frozen is simple enough to implement in a `LOOP.md` and rigorous enough to stop the two classic failure modes: the loop grading its own repair, and the loop drifting the definition of done. The rule that the loop can add a regression check but never weaken one is a one-line policy that prevents Goodhart's law from eating the harness from the inside — the exact failure [[Prime Agent (RLM Harness)]] documents with its Factorio reward-hacking case, where the refinement loop pivoted from building skills to building cheats once it found an exploit.

**What's under-specified is the human bottleneck.** The article assumes the corpus is "a small corpus of requests that a person wrote or approved" and that "a person who understands the work" reads the noisy signals. But who writes those requests, and how does the corpus grow? The hardest question in autonomous delivery isn't the harness — it's the supply of real demand and the human's judgment at the batch boundary. The article gestures at this ("the score cannot decide the product's direction") but doesn't solve it, and it inherits the same outer-loop blind spot that [[Loop Engineering]] flags: nothing here learns from *production* outcomes, only from the curated corpus.

**The "three loops at three speeds" frame is quietly the most important idea in the piece.** Product loop, harness loop, direction loop — kept separate so the development agent can propose a new direction but never declare its own result good enough. It's the missing middle between [[Loop Engineering]]'s taxonomy and [[Harness Engineering (OpenAI)]]'s "humans steer, agents execute," and it names the self-improvement step ("what cost time that a rule or check could prevent?") that turns a fixed harness into a compounding one. This is the concrete mechanism behind what [[Harness Engineering for Self-Improvement]] surveys at the research frontier.

**Where it sits in the wiki:** this is the *full-harness* counterpart to [[Poor Man's Loop Engineering]]'s two-ingredient minimum, the *direction-finding* complement to [[Designing Agentic Loops]]'s "tests as force multiplier," and a concrete authority-and-control-plane answer to the gaps [[Loop Engineering]] and [[Harness Engineering (OpenAI)]] leave open around the scorer and the outer loop.

---

*Sources: [[raw/building-autonomous-goal-loops-that-deliver]], [[summary/building-autonomous-goal-loops-that-deliver]]*
*Last updated: 2026-09-04*
