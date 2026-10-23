# AI Agent Teams That Don't Loop (AWFlow)

AWFlow's write-up of the coordination runtime behind its multi-agent teams: the argument that free-for-all agent chatrooms loop and burn tokens in production, and that the fix is structure enforced in code — snapshot-frozen team definitions, a handoff map that literally removes the delegation tool when no target is permitted, budgets checked before each handoff, and a pure-orchestration core testable without a model.

---

## What it argues

Multi-agent demos show agents chatting in a shared room until something good emerges. In a product that yields loops, duplicate work, agents answering questions nobody asked, and token bills that grow with every round. AWFlow's team runtime replaces the chatroom with a way of working: three topologies (lead-routes, pipeline, open-table), a snapshot of the team frozen at thread start, and every constraint enforced in code rather than requested in prose.

## Key quotes

> "An agent that the map gives no one to hand to doesn't even see the tool. That's cheaper and more reliable than telling the model in a prompt 'please only delegate to X'."

The sharpest design idea in the piece: make the disallowed action structurally impossible instead of politely prohibited. This is the tool-shaping version of what harness engineers keep rediscovering — capability boundaries belong in the environment, not the system prompt.

> "All of it is enforced in code around each tool call. It isn't a paragraph in the system prompt, so a web page can't talk an agent out of it."

Guardrails phrased as prompt text are suggestions; phrased as code around the tool call they're physics. The injection-resistance framing ("a web page can't talk an agent out of it") is exactly right — rules that survive hostile input are the only rules that count.

> "The thread can't run forever because the check comes first."

Budgets (steps, handoffs, cloud tokens) are checked *before* spending, and the result is kept even when it exhausts a limit — a small but mature detail: the last useful handoff isn't discarded for exceeding its allowance.

> "You can reason about coordination without ever calling a model."

Injecting `runMember` so the whole runtime unit-tests with fake agents is presented as "the single best decision you can make" if building something similar. It separates orchestration logic — modes, maps, budgets, concurrency, aborts — from model behaviour, which is the only way such logic can be deterministic enough to trust.

> "Asks are one level deep, so they never loop."

The recursion cap on `ask_teammate` is the whole anti-loop story in one line: delegation graphs that can recurse are graphs that can spin.

## Themes

#concept #pattern #tool

## Analysis

This is a refreshingly unglamorous piece. While much multi-agent writing chases emergent swarms, AWFlow's thesis is that teams become usable precisely when you *remove* freedom: fixed topologies, frozen snapshots, map-constrained delegation, one-level asks, hard budgets. The pipeline mode's pitch — "friendliest for small local models" — is an honest admission that coordination overhead is a tax the weakest model in the team must be able to pay.

The snapshot rule is under-discussed elsewhere: editing a team mid-thread silently changing the rules under a running conversation is a real footgun, and resolving it by asking whether running threads keep the snapshot or adopt the change is the right kind of boring. The weakest point is the browser runtime itself — threads live only while the browser is open, and the "honest limit" section barely gestures at what that costs. But the coordination patterns are tool-agnostic and travel well.

## Relations

- [[Agent Orchestration for the Timid]] shares the thesis that orchestration should constrain rather than liberate agents; AWFlow supplies the concrete runtime mechanics (maps, budgets, snapshots) behind that caution.
- [[Multi-Agent Orchestration Across Repositories]] operates at a different scale (repositories, not tasks), but both treat delegation as a designed graph rather than free chat; AWFlow's "hand off the task, not the transcript" nuance is a context-management lesson for any delegation topology.
- [[A Convention Is Not a Constraint]] makes the same prose-vs-code distinction AWFlow leans on: rules that only live in a prompt can be talked out of, so the enforcing layer must be deterministic. This source is a worked example of that principle applied to multi-agent delegation.
- [[Anthropic's Advisor Strategy]] is the small-scale counterpart — a two-agent orchestrator-worker relationship from a major lab — while AWFlow generalises to arbitrary team shapes with merge-only-stricter rules.

---
*Sources: [[raw/ai-agent-teams-that-dont-loop]], [[summary/ai-agent-teams-that-dont-loop]]*
*Last updated: 2026-10-24*
