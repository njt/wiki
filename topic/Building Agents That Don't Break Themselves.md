# Building Agents That Don't Break Themselves

Daniel Botha's tight 7-minute essay on the single most important architectural decision in agent design: separating where an agent *lives* from where it *acts*. Through two contrasting Fly.io Sprite case studies — one ephemeral, one persistent — Botha makes the case that sandboxing isn't a security add-on but the frame on which agent reliability hangs.

---

## Key Quotes

> "Where your agent lives and where it runs code are two entirely separate considerations."

The thesis, stated cleanly. Not a novel insight — containers have been saying this for a decade — but Botha applies it specifically to the agent loop/harness split in a way that the agent community has been slow to internalize. Most agent frameworks still treat isolation as "run the whole thing in Docker" rather than "give the brain a comfortable home and the hands a padded room."

> "The agent's home can be durable and comfortable. The place it runs untrusted strings should still be somewhere you would be happy to set on fire."

This is the practical corollary. A durable agent home (with persistent memory, identity, installed tooling) doesn't require durable execution environments — it requires the opposite. You want the execution surface to be so disposable that you *can* checkpointer `rm -rf /usr/bin/python3` and be back in nine seconds.

> "Checkpointing before every risky step is cheap enough to be a reflex."

The concrete implementation detail that makes the philosophy actionable. Copy-on-write checkpointing turns the risk calculation from "will the agent break itself?" to "do I mind losing the last 500ms of work?" The latter is always no. This is the same insight as [[Postgres Transactions Are a Distributed Systems Superpower]] applied to agent execution. At cluster scale, [[Agent Substrate]] generalizes this pattern: its control plane suspends idle agents to disk and resumes them on any available worker in under a second, making checkpoint/restore the core scheduling primitive rather than a safety net.

> "Telling your agent to be careful is silly. Just make it do things somewhere it doesn't have to be."

The aphorism that closes the piece and names the core error in most agent safety thinking: treating safety as a prompt-engineering problem rather than an architectural one. This lands in the same intellectual territory as [[How We Contain Claude]]'s finding that 93% permission prompt approval rates make prompt-level safety a consent theater, not a security boundary.

> "The sandbox is the security boundary now."

Botha's observation about the Hermes agent skipping confirmation prompts. Once commands run in a true sandbox, the security decision shifts from "should I allow this?" (human judgment, unreliable under fatigue) to "will this damage anything I care about?" (architectural constraint, reliable regardless). Same conclusion [[Security and Sandboxing]] arrives at from the container-security direction.

## Case Studies

**SpriteDoc** (by Henrique) is the ephemeral model: each user session gets its own sandbox, secrets injected per-command and never stored at rest, the sandbox destroyed when the session ends. The clean-room model — nothing persists, nothing leaks.

**Hermes Agent** (by Kyle, for Nous Research) is the persistent model: one sandbox per task, resumed across sessions so tooling installations survive. Because isolation is guaranteed, dangerous commands are allowed — no confirmation prompts needed.

The contrast is productive: Botha doesn't pick a winner. Ephemeral sandboxes are simpler and safer. Persistent sandboxes are more practical for long-running tool chains. The principle they share — execution happens elsewhere — matters more than the lifecycle choice.

## Critical Analysis

**The strongest claim is the one Botha doesn't belabor**: defense in depth applied to sandbox nesting. Running an agent inside a sandbox that dispatches commands to a *different* sandbox — verified by Kyle with Sprite ID mismatches on return — means an agent can't compromise its own execution environment even if it *tries*. This is the architectural equivalent of the principle that [[How We Contain Claude]] lands on after three years of incidents: "the standard primitives held while our own work around them exposed flaws." A sandbox you own and maintain is less trustworthy than a sandbox you provision and discard.

**What's missing**: Botha doesn't address credential management for persistent sandboxes. The Hermes approach of keeping installations across sessions implies the agent has long-lived access to *something* — package registries, API endpoints, git repos. SpriteDoc's per-command token injection solves this for ephemeral use, but the persistent model needs a complementary story. [[OneCLI]]'s transparent credential injection and [[You Dont Want Long-Lived Keys]]'s ephemeral credential principle are the missing chapters here.

**The checkpointing economics are under-explored.** Botha says copy-on-write makes checkpointing cheap, which is true for filesystem state, but what about network effects? If the agent has already posted to Slack or pushed to a remote, checkpoint rollback can't undo those external side effects. The undo button is real for local state and illusory for distributed state — a tension that [[The Agentic Product Standard v2.0]]'s execution harness design grapples with but Botha leaves unaddressed.

**The editorial omission**: Botha works at Fly.io and the entire piece is built around Fly.io's Sprite VMs, but the design principles are vendor-neutral. The "padded room" metaphor and the brains/hands split apply to any container/VM isolation: Docker, Firecracker, gVisor, Incus, even macOS Seatbelt. [[Best Infrastructure Platforms for Coding Agents in 2026]] surveys the field Botha is implicitly critiquing — most platforms offer one sandbox per agent, not the nested sandbox pattern he argues for.

## Key Themes

#agent-architecture #sandboxing #security #checkpointing #defense-in-depth #agent-reliability

## Connections

- [[Security and Sandboxing]] — the hub page that collects the isolation landscape Botha is contributing to. His nested-sandbox argument is a missing section in that page's "What's Missing" list.
- [[How We Contain Claude]] — Anthropic's postmortem-adjacent containment tour validates Botha's thesis from the other direction: every incident was in custom code around standard sandbox primitives, never in the primitives themselves. Botha's answer is to make the custom code part *also* run in a throwaway sandbox.
- [[Hermes]] — the actual project that Kyle built for Nous Research, described in Botha's second case study. Botha's description of Hermes using persistent sandboxes to skip confirmation prompts isn't in the Hermes README — it's architectural context only visible from the infrastructure side.
- [[Best Infrastructure Platforms for Coding Agents in 2026]] — the sandbox platform landscape. Botha's piece reads as an implicit design brief for what the next generation of these platforms should support: nested sandboxes, not just sandbox-per-agent.
- [[Components of a Coding Agent]] — the harness architecture taxonomy. Botha's brains/hands split maps cleanly to the harness/execution boundary in that taxonomy.
- [[Coding Agents Continuity Not Memory]] — Santi's continuity-as-primitive thesis. Botha's durable-agent-home + disposable-execution-surface split is the architectural answer to "give me continuity without giving me an execution surface I can break."
- [[The Agentic Product Standard v2.0]] — the execution harness design in that standard would benefit from Botha's checkpointing reflex as a first-class primitive.

---
*Sources: [[raw/building-agents-that-dont-break-themselves]]*
*Last updated: 2026-07-08*
