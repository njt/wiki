# Agent State Machines (Josh Rosen)

Josh Rosen's taxonomy of agent state machines: explicit states, events and transition rules that take the model out of the driver's seat. His thesis is that anything important enough to matter — approvals, recovery, validation, multi-session work — should be enforced by software, not asked of the model in a prompt. He maps four layers where these machines live (inside the loop, over workflows, supervising many agents, and on the work itself), surveys real implementations (Stately Agent/XState, LangGraph, Mastra, n8n, OpenAI Symphony, Pi Missions, ThruWire), and explains why developers keep building their own despite mature frameworks.

---

## Key Quotes

> "If you're not using some type of agent state machine, you're completely at the mercy of the nondeterministic nature of models. Are you really okay with models deciding what actions to take and when their work is complete?"

The framing question of the piece, and a sharper version of the guardrails argument: prompts and skills leave operational rules to the model's discretion; a state machine moves them out. "The model can request an action or propose a transition, but the surrounding software determines whether it is permitted."

> "Encoding these requirements entirely in prompts or skills leaves the model responsible for following operational rules that could instead be enforced by software."

This is the same insight as [[A Convention Is Not a Constraint]] and Anthropic's advisory-skills-vs-deterministic-hooks split, arrived at from the architecture side rather than the process side: anything the loop must obey belongs in the loop's skeleton, not its instructions.

> "The model writes the response, but the application controls which transitions are permitted."

Rosen's one-line summary of the whole pattern, from the Stately Agent customer-service example. The division of labour is strict: generation to the model, permission to the program.

> "These checkpoints are fundamentally different from execution checkpoints in systems like LangGraph... LangGraph saves execution state so an interrupted process can resume. ThruWire maintains the state of the work itself, including its requirements, dependencies, and validation evidence."

The sharpest distinction in the essay. Execution checkpoints make a *process* resumable; work-state machines make the *work* acceptable. The second is where [[Specifications as the Product]] thinking and verification evidence converge: the machine's state IS the spec's completion state.

> "Developers who start with simple state machines can eventually find themselves implementing checkpointing, distributed locking, scheduling, observability, and crash recovery. At that point, they may have rebuilt much of the infrastructure they initially avoided."

An honest statement of the homegrown trap, and the best reason to reach for LangGraph-style runtimes even when five bespoke states look sufficient.

## Key Themes

- #concept — deterministic scaffolding: transitions enforced by code, decisions delegated to the model
- #pattern — the four layers: in-loop, workflow graph, cross-agent supervisor, work-state machine
- #tool — XState, LangGraph, Mastra, n8n, Temporal/Restate/DBOS for durability
- #person — Josh Rosen, researching agent architectures and building ThruWire

## Analysis

The essay's real contribution is the layering argument. Most state-machine discussions collapse into "use LangGraph or not"; Rosen shows these machines appear at four distinct levels and — crucially — that they *compose*: an in-loop machine, a cross-session supervisor, and a work-acceptance system can all coexist over the same agent. That framing dissolves the false choice between "trust the model" and "script everything": you script the boundaries you can afford to be wrong about least, and leave the how to the model.

The taxonomy of hand-rolled implementations is equally useful. The three integration routes — SDK/subprocess orchestration, harness extension tools, and an MCP server exposing `claim_task`/`complete_task` — are effectively a design space map, and the MCP option is the most interesting because it decouples the state machine from any single harness.

The ThruWire section, where the author discloses his own product, is where the essay is weakest on critical distance — the work-state machine layer gets the most novel framing and the least independent evidence. And the piece is light on failure cases: what does an over-constrained state machine cost when the model's judgement was actually better than the transitions? The honest answer is probably "rarely, and worth it," but the essay never asks.

Still, as a map of where determinism lives in agent infrastructure, this is the clearest short treatment in the wiki's orbit.

## Related

- [[Dmitry Sotnikov's LLM Workflow]] — strengthens Rosen's thesis from the practitioner side: Sotnikov independently arrived at "routing logic should be elevated to first class citizenship... state machines are the natural fit," with the model demoted to a leaf node in deterministic control flow.
- [[Fleet Supervisor (sermakarevich)]] — a concrete homegrown instance of Rosen's supervisor layer: claim/spawn/reap/retry/block loops as transition functions over a task queue, with the supervisor deliberately dumb and the model deciding how.
- [[Procedural Graphs]] — complicates the picture from the research side: rather than hard-coding transitions, an explicit editable graph supplies step-local guidance while leaving generation free — a softer alternative to enforcing transitions in code.

---
*Sources: [[raw/2117687393213932003]], [[summary/2117687393213932003]]*
*Last updated: 2026-11-06*
