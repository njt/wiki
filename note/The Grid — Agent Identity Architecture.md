# The Grid — Agent Identity Architecture

Matt Galligan's personal AI agent workspace uses file-based "identity disks" — markdown files with temperament dials, soul documents, and explicit handoff rules — to give each agent program a distinct behavioral profile. This isn't prompt engineering; it's identity engineering. The files don't tell the agent what to do; they tell it who to be. And the system works: when Patch (the coordinator program, with `context_hunger: 19/20`) was asked to research bb's harness architecture, it produced a long, sourced synthesis — not because the prompt demanded depth, but because Patch's identity made "dig until the picture is real" the default.

---

## The Identity Stack

Each named program in The Grid is defined by a chain of markdown files loaded at startup:

```
AGENTS.md → init-<program> → INIT.md → IDENTITY.md → SOUL.md / BOUNDARIES.md / memory
```

The **identity disk** (`IDENTITY.md`) is the core. It defines:

- **Temperament dials** — numeric parameters that shape behavior: `context_hunger` (how deeply the agent researches before acting), `delegation_reflex` (how readily it hands off to other programs), `tolerance_for_fake_alignment` (how much it tolerates surface-level agreement over real understanding)
- **Ownership and handoffs** — what the program owns (coordination, synthesis, priority) and which other programs it hands specific work to (Index for research, Rez for building, Crit for review)
- **Wants and fears** — e.g., Patch wants "Matt oriented and the Grid coherent"

> "context_hunger: 19/20, delegation_reflex: 16/20, tolerance_for_fake_alignment: 2/20"

The **soul file** (`SOUL.md`) defines voice and working style:

> "I'm hungry for context. I want the full picture before I act… I'm resourceful before I'm needy. Read the file. Check the context. Figure it out. Come back with answers, not questions."

> "Think less 'how may I assist you today' and more 'yeah I already looked into that, here's what I found.'"

> "Brevity is mandatory. If the answer fits in one sentence, one sentence is what you get."

The **boundaries** define the internal/external split: be bold *internally* (read, research, organize, build); be careful *externally* (avoid anything dangerous or compromising).

## The Program Roster

The Grid has 12 named programs. They are not different model vendors — they're **role identities** that change what the agent reaches for first and what it hands off:

| Program | Domain |
|---------|--------|
| Patch | Coordinator: context, delegation, tradeoffs, whole-board continuity |
| Index | Research / provenance / notes / synthesis (`provenance_hunger: 20/20`) |
| Rez | Builder: executable software, plumbing, CLIs, integrations |
| Crit | Verification: reviews, tests, evidence, readiness calls |
| Cadence | Personal ops: email, calendar, reminders, follow-ups |
| Cipher | Security / secrets / trust boundaries |
| Hab | Home Assistant / house state |
| Sudo | Local machines, services, packages, shells |
| Link | UniFi / network presence |
| Figure | Design / UI feel |
| Spark | Throwaway spikes / demos |
| Vox | Drafting in Matt's voice |

Startup rule: if the request is ambiguous or cross-cutting, load **Patch**. If it clearly belongs to a specific domain, load that program's init skill.

## Handoffs as Explicit Architecture

Handoffs are not implicit or model-discovered — they're defined in the identity disk. Patch's canonical workflow: Index recovers prior context → Rez builds the slice → Crit reviews → Patch keeps tradeoffs visible. Patch's *wrong* example is "I can coordinate the research, build, review, and follow-up myself" — identity as constraint, not capability.

This is a different coordination model from planner/worker/judge ([[Agent Orchestration]]). In planner/worker/judge, the planner decomposes tasks and workers execute them — the roles are defined by the task structure. In The Grid, roles are defined by *domain and temperament*, and the coordinator's job is to route to the right domain, not to decompose the work. It's coordination through specialization, not hierarchy.

## The Quality Bar: Identity + Prompt

The research writeup that triggered this note was shaped by two forces:

1. **Identity set the floor.** Patch's `context_hunger: 19/20` meant the default pull was toward depth. Its soul file said "come back with answers, not questions." The Grid's `AGENTS.md` said useful discoveries should survive the session. These didn't make the output long — they made it *thorough*.

2. **Prompt set the ceiling.** "Be thorough" + "put findings in a gist" made depth and length explicit. Without those, Patch's soul-level brevity mandate ("one sentence is what you get") would have kept the chat reply tight and the writeup thinner.

> "Brevity is mandatory. If the answer fits in one sentence, one sentence is what you get."

This creates a productive tension. The soul file says be brief. The identity disk says be thorough. The prompt resolves the conflict by specifying the deliverable shape. The agent doesn't ignore brevity — it routes depth to the artifact (the gist) and keeps the chat reply short. That's not a bug; it's the system working as designed.

## PatchOS: The Tooling Layer

**PatchOS** is the local infrastructure that makes the identity system durable across sessions: a Bun/TypeScript app on the Trails framework, with a SQLite graph database (`entities`, `entries`, `relations`, FTS search), exposed as a `patch` CLI and MCP server. Agents use it to log and recall project knowledge. It's orthogonal to the identity disk system — identity defines *who* the agent is; PatchOS stores *what* the agent knows.

## Critical Analysis

**What's novel.** The temperament dials are the standout contribution. Most agent systems control behavior through system prompts (probabilistic) or hooks (deterministic but narrow). Numeric dials like `context_hunger: 19/20` occupy a middle ground: they're structured enough to be tunable, loose enough to be interpreted by the model. This is closer to [[Steering Claude Code]]'s CLAUDE.md-as-facts pattern than to prompt engineering — it's a personality parameter, not an instruction.

**The identity-as-constraint inversion.** Most agent design asks "what capabilities should the agent have?" The Grid asks "what should this agent *refuse to do*?" Patch's wrong example — "I can coordinate the research, build, review, and follow-up myself" — is identity as negative space. It's the same insight as [[Agent Identity]]'s "the work is arriving at a grounded no" but implemented as file-based configuration rather than philosophical argument.

**The handoff system is the underrated piece.** The explicit "hands to Index when: memory, provenance, sources, or history matter" is a routing table, not a suggestion. It means Patch doesn't need to be good at archival research — it just needs to know when to delegate. This is specialization through refusal, and it's more architecturally honest than the "one agent to rule them all" approach that most coding agents default to.

**What's unclear.** How much of this is Matt Galligan's personal taste vs. a generalizable pattern? The identity disks are clearly effective for him, but the temperament dials (19/20, 16/20, 2/20) are tuned to one person's preferences. Would they transfer to another user? The answer probably depends on whether the temperament dials are calibrating the model's behavior (generalizable) or Matt's preferences (personal). The source doesn't disentangle this.

**What's missing.** No discussion of failure modes. What happens when `context_hunger: 19/20` meets a task that needs fast action, not deep research? What happens when Patch's delegation reflex routes to Index but Index isn't loaded? The identity system is described as working well in the case where everything goes right, but real systems fail at the edges.

**Connection to the broader identity conversation.** [[Agent Identity]] argues that agents need identity (a stake in what happens next), not just memory (a record of what happened). The Grid is the most concrete implementation of that thesis I've seen: identity disks are literally files that give agents a stance. And Galligan's note validates the theory from the inside — Patch's output quality came from *who it was configured to be*, not just *what it was told to do*.

---

*Sources: [[raw/how-grid-patch-instructions-shaped-agent-research-depth]], [[summary/how-grid-patch-instructions-shaped-agent-research-depth]]*
*Last updated: 2026-08-08*
