---
url: https://awflow.io/blog/ai-agent-teams-that-dont-loop
date_fetched: 2026-10-24
---

“Multi-agent” demos usually show a few agents chatting in a shared room until something good comes out. That works in a demo. In a product it gives you loops, duplicate work, agents answering questions nobody asked, and token bills that grow with every round.

AWFlow (a browser extension for automation) recently shipped agents and teams of agents. Each agent has its own model, instructions, memory, skills and rules. This post is about the part that made teams usable: the coordination runtime. If you want the background first, two earlier posts cover what a workflow really is and how agent automation works: an agent is 5 parts, a team is 3 more.

A way of working, not a chat room

Free-for-all coordination is the default in most demos, and it is the source of the problems above.

A team is a snapshot

When a team thread starts, it takes a snapshot of the team: its members, their roles, the way of working, the handoff map, its rules and its limits. Everything the runtime decides is based on that snapshot, so editing the team can’t silently change the rules under a running conversation. New threads use the new setup, and AWFlow asks whether running threads keep the snapshot or apply the change.

Three ways of working

Every team picks one way of working. The first one is the classic “manager” shape.

The lead plans

In lead routes, only the lead’s chat turn gets a delegate_to(agent, task) tool. It plans the job and decides who does what.

Members do their part

Each member receives a task, works with its own model, skills and rules, and returns a result.

Results go back to the lead

Nobody else sees the exchange as an open conversation. The result goes back to the agent that asked.

The lead merges

The lead turns the parts into one answer. This mode is best for open-ended goals like “plan a weekend in Lisbon”.

The second way of working removes planning altogether. In a pipeline, members run in a fixed order and each receives the previous result. No agent has to plan. It’s best when the steps are known but each needs judgement (research, then write, then repurpose), and it’s the friendliest mode for small local models.

The third, the open table, is for when a human is steering. Whoever you @mention answers (or the lead, if nobody is mentioned), and can hand off from there.

Handoffs follow a map

delegate_to isn’t “message anyone”. It’s generated per agent from the team’s handoff map.

In code, the tool only exists when the map gives an agent someone to hand to:

/** `delegate_to` for an agent, or nothing when the map gives it no one to hand to. */function delegateTools(fromId) { const targets = targetsOf(fromId); // only the teammates the map allows if (!targets.length) return []; // no targets → no tool at all // …describe the allowed targets in the tool's schema}

An agent that the map gives no one to hand to doesn’t even see the tool. That’s cheaper and more reliable than telling the model in a prompt “please only delegate to X”. When a model writes @writer or “Writer”, the runtime resolves the reference against the team’s members (by id, handle or name), and anything else simply isn’t a valid target.

Hand off the task, not the transcript

A handoff carries the task and the context it needs, not the whole conversation. With small local models (2–4k token windows) that’s the difference between working and not working. With cloud models it’s the difference between a cheap thread and an expensive one.

Budgets and rules, enforced in code

Every thread has three limits: steps, handoffs and cloud tokens. Every handoff is checked against them before it runs:

async function handOff(from, to, task, context) { check({ handoffs: 1, steps: 1 }); // refuse before spending anything usage.handoffs++; const result = await gate(() => // concurrency gate (parallel teams) runMember(to, task, { from, context, tools: delegateTools(to.id), signal, onCloudTokens })); count(result); // steps + cloud tokens actually used check(); // the result is kept even if it used up a limit return result.reply;}

Seen as a cycle, each handoff is a lap: check, hand off, count. The thread can’t run forever because the check comes first.

The details that matter in practice:

A team can only tighten the rules

Each agent has rules: allow, ask or block per action, plus locked categories like paying, sending, deleting or entering credentials. In a team, the team’s rules are merged so they can only make a member stricter. A permissive agent joining a careful team becomes careful, never the other way around.

All of it is enforced in code around each tool call. It isn’t a paragraph in the system prompt, so a web page can’t talk an agent out of it.

Asking a teammate can’t recurse

Outside team threads, an agent can have a “Teammates” skill: ask_teammate(teammate, task) runs the other agent headless on its own model, skills and rules, and returns the answer. One rule keeps this sane: a teammate that’s being asked can’t ask teammates in turn.

Asks are one level deep, so they never loop.

Testable by design

The coordinator is pure orchestration. Running one member is injected (runMember), so the whole runtime (modes, map, budgets, concurrency, aborts) is unit-tested with fake agents. The real member runner (model, skills, memory, rules, browser tab group) lives elsewhere.

If you build something similar, this split is the single best decision you can make: you can reason about coordination without ever calling a model.

Workflows and teams aren’t rivals

AWFlow started with deterministic workflows: draw the path, trigger then steps then output. Teams are the other way to automate: describe the outcome and let agents find the path. They combine. An agent can run a workflow as a skill when it needs a reliable step, and a workflow can call an agent (the “Run agent” node) when it needs judgement. An agent’s scheduled routine is itself an ordinary workflow you can open and edit. The teams page in the docs walks through setting one up, and the agents page shows what an agent can do.

The honest limit: agents and teams run in your browser, so a thread works while the browser is open, and local models are smaller than the frontier cloud ones. With a small local model, the pipeline mode is the one to start with.

AWFlow is free and runs in your browser. Agents run on the model you pick: in-browser, Ollama, or your own API key.
