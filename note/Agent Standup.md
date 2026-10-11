# Agent Standup

Agent Standup is a free, open-source tool that turns a Claude Code session into a live "standup meeting" visualization: the lead agent as the meeting's chair, sub-agents joining, sharing screens, reporting one status each, and leaving — with voices, characters, replay, and a context/token gauge. The useful part hides inside the joke: "see who is waiting on whom and who is blocked, at zero token cost."

---

The framing is deliberately comic — a fake standup call where your sub-agents are the attendees — but the underlying information design is serious. Multi-agent Claude Code runs are hard to observe from a terminal scrollback: which sub-agent is running, what it is blocked on, how much context each has burned. Agent Standup surfaces exactly that as one status per agent, a context gauge, token counts, and a replay/recap/tree view, served from one small local server reachable via VSIX (Cursor/Windsurf/VSCodium), any terminal, a desktop app, a Claude Code status line, or tmux. Prerequisite: you've used Claude Code on that machine.

The author is disarmingly honest about scope. "Is it serious?" — "Mostly for fun. No." — signed "Made by a bored dev and his robots." That honesty is refreshing next to agent-infra marketing that oversells thin tooling.

> "See who is waiting on whom and who is blocked, at zero token cost."

The key claim: observability of the agent tree should cost no tokens. Everything here is a read-side view over state Claude Code already tracks locally, not extra model calls. That's the right economics for harness tooling — instrumentation that pays inference rent rarely survives.

> "It's a bit silly. It's also weirdly useful."

A good test for agent UX: if the metaphor makes the state legible, it doesn't matter whether the metaphor is a video call. Compare [[Television (Agent GUI)]], which argues agents need persistent pinned artifacts rather than chat; Agent Standup makes the same argument for the *orchestration* layer — the run's structure deserves a persistent, glanceable surface too.

## Themes

- #tool — a local, open-source Claude Code companion, multi-surface by design
- #concept — zero-token observability of agent runs
- #pattern — anthropomorphizing orchestration state (a standup call) to make it legible
- #concept — agent UX as a first-class problem, not an afterthought

## Take

The standup framing is a gimmick with a real product inside it. The interesting artifact is the data model implied by the feature list: per-agent status, waiting-on relationships, context consumption, a replayable tree. That is the observability layer the orchestration tooling ecosystem keeps lacking — and here someone shipped it as a joke. The honest "no, it's not serious" positioning is itself a small critique of the space: genuinely useful agent tooling can arrive dressed as a toy when the "serious" version is still being pitched.

## Related pages

- [[Television (Agent GUI)]] — makes the same agent-UX argument one layer up: agent output deserves persistent, structured surfaces rather than chat scrollback; Agent Standup applies that insight to watching the multi-agent run itself.
- [[Docker Agent]] — a far heavier take on multi-agent teams and delegation; Agent Standup is the observability counterpart to Docker's YAML-declared orchestration, showing what a run looks like from outside.
- [[How Boris Uses Claude Code]] — a practitioner's view of driving Claude Code sessions; Agent Standup adds a live spectator layer to exactly those sessions.

---
*Sources: [[raw/agent-standup]], [[summary/agent-standup]]*
*Last updated: 2026-10-11*
