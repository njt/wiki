# Sidekick — Persistent Worker Agent Skill

A Claude Code skill from jleechanorg's claude-commands collection that spawns persistent, crash-recoverable sidekick agents for long-running missions. It's the most thorough public write-up of the "durable agent teammate" pattern for Claude Code: STATE.md checkpoints as the durability primitive, tmux as process persistence fallback, and a stall watchdog born from real production incidents.

---

## Key Quotes

> "Interactive TUI sidekicks fail by **stalling alive**, not by exiting."

This is the document's sharpest observation. The session-limit modal — that Claude Code popup asking whether to continue — blocks all work silently. An agent that hasn't crashed and hasn't finished is the hardest failure mode to detect: the process is running, the pane is visible, nothing is happening. The 15-minute STATE.md mtime watchdog is the diagnostic.

> "Auth env propagation is MANDATORY in the launch command. Never trust the tmux server to pass them."

A hard-won lesson from 2026-07-10 incidents. The tmux server snapshots environment at startup, so environment variables set in your shell after tmux launches don't reach new panes. The fix is bruteforce: inline everything into the tmux command string. Not elegant, but it's the only thing that works.

> "Team visibility mandate: The user always wants the sidekick visible in the invoking session's Agent Team panel."

The tension between durability and visibility defines the entire architecture. In-session teammates are visible but die with the parent session. Tmux sidekicks survive but are invisible to Agent Teams. The hybrid pattern — in-process by default, tmux as fallback — is the pragmatic resolution. This is a real product constraint, not an abstract architecture preference.

> "Endgame single-writer freeze: When a lane reaches the final implementation stage, only one writer may touch the target files."

From a PR incident retro, this is the hardest drive-loop invariant. Multiple agents writing the same files in parallel creates merge conflicts that cascade. The fix is organizational, not technical: freeze writes when one lane enters endgame. It's the kind of rule you only write down after losing an afternoon to a merge nightmare.

## Key Themes

### Checkpoint Durability as Architecture (#pattern)

The central insight: STATE.md is not a log, it's a resumption protocol. Five components (Mission, Ground Truth, Standing Rules, Progress Log, Next Actions) where the last one — rewritten every step — is the load-bearing piece. A fresh session reads Mission and Next Actions and resumes without reading the full history. This is the same pattern as database write-ahead logs and event sourcing, reduced to markdown.

The 5-minute checkpoint cadence makes the tradeoff explicit: "a crash loses ≤5 min." That's a service-level objective stated as a design constraint.

### Stall Detection Is the Hard Problem (#tool)

Crashes are easy — the process exits, you notice. Stalls are hard — everything looks alive, nothing moves. The sidekick skill diagnoses the specific failure mode (session-limit modal blocks work), gives a detection mechanism (STATE.md mtime >15 min), and prescribes a recovery (`tmux send-keys Enter`). This is operational knowledge you can't get from API docs.

The corollary — "check EVERY teammate pane, not just the lead" — is the kind of rule that reads as obvious in hindsight and was probably learned the hard way.

### The Visibility-Durability Tradeoff (#concept)

Agent Teams is one-team-per-session with no cross-session join. This is the architectural constraint that forces the two-mode design. In-process teammates are visible and SendMessage-addressable but transient. Tmux sidekicks are durable but invisible and reachable only through filesystem state + tmux capture-pane. Neither satisfies all requirements, so the skill layers both.

This is not a Claude Code limitation — it's a genuine distributed systems problem. A durable agent outlives the connection that spawned it, and no in-band protocol can reach it. STATE.md becomes the out-of-band channel.

### Skills as Operational Runbooks (#pattern)

The sidekick SKILL.md reads less like API documentation and more like an SRE runbook. It has incident timelines (2026-07-10, 2026-07-11), failure mode catalogs, watchdog procedures, and migration playbooks. The drive-loop invariants are postmortem action items encoded as standing rules. This is what production-grade agent infrastructure looks like when someone writes down what actually broke.

## Critical Analysis

**The document's real contribution is the failure modes, not the happy path.** Anyone can write "spawn a tmux session and run Claude Code." The stall watchdog, the auth propagation bug, the endgame single-writer freeze, the team visibility constraint — those are what make the skill valuable. This is a postmortem disguised as documentation, and it's better for it.

**STATE.md as a protocol is under-specified.** The document says "append timestamped heartbeat" and "rewrite Next Actions each step" but doesn't specify the format tightly enough for a machine to parse reliably. A JSON or YAML state file with a defined schema would make the resumption bead programmatic rather than human-mediated. The markdown format prioritizes human readability, which is the right call for a v1, but the next step is a structured state format.

**The two-mode architecture is honest about its compromises.** Rather than papering over the visibility-durability gap with abstraction, the skill makes the tradeoff explicit: here's when you get visibility, here's when you get durability, here's how to migrate between them. That honesty is more useful than a clean API that hides the failure modes.

**The watchdog should be a separate process, not a supervisor responsibility.** Requiring the human supervisor to poll STATE.md mtime is fragile — it's exactly the kind of background monitoring that humans are bad at. A watchdog daemon (or a cron job, or a Claude Code hook) that alerts on staleness would close the loop. The document diagnoses the problem perfectly but leaves the solution as a manual procedure.

**This skill and Fleet Supervisor solve complementary halves of the same problem.** Fleet Supervisor (sermakarevich) handles the orchestration — spawning parallel coders, atomic task queues, web UI. Sidekick handles the durability — crash recovery, state checkpoints, stall detection. A merged system that used Fleet Supervisor's orchestration with Sidekick's durability patterns would be more capable than either alone.

**The "auth env propagation" fix is fragile.** Inlining environment variables into the tmux command string works but leaks secrets into `ps` output and shell history. A proper solution would use tmux's `set-environment` or a sidecar that injects environment after pane creation. The current approach is correct about the problem but incomplete as a solution.

**The skill is really about making agents survive the session boundary, and that's the right problem.** Most agent infrastructure focuses on what happens during a session — tool calling, context management, prompt engineering. The sidekick skill focuses on what happens between sessions — state persistence, resumption, failure recovery. That boundary is where the hard distributed systems problems live, and it's where most agent tooling is weakest.

## Related Pages

- [[Fleet of Agents (sermakarevich)]] — Five-step tutorial building from a single Claude Code agent to a parallel fleet with atomic claiming. The "filesystem as memory" pattern in Step 1 is the same primitive Sidekick's STATE.md builds on, and the Q&A blocking in Step 5 solves the same stall problem.
- [[Fleet Supervisor (sermakarevich)]] — Production Python supervisor with six outcome classifications, context-pressure enforcement, and five concurrent asyncio control loops. Complementary: Fleet Supervisor handles orchestration at scale, Sidekick handles crash recovery and durability for individual agents.
- [[Mission Control — Bhanu's 10-Agent Squad on OpenClaw]] — The best published reference for a persistent multi-agent team: 10 specialized agents on 15-minute heartbeat loops, file-based memory (WORKING.md maps directly to STATE.md), and Kanban-based team visibility. Sidekick provides the Claude Code-native version of the same architecture.
- [[Planning With Files]] — Claude Code skill implementing a three-file persistent state system with hook-enforced plan re-reading. 96.7% pass rate with the skill vs. 6.7% without — the quantitative proof that structured file-based state works. The purest minimal form of Sidekick's STATE.md checkpoint pattern.
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — Santi's thesis that context ≠ continuity and the real primitive is repo-local, evidence-weighted state with a resume-work-finalize lifecycle. Sidekick's STATE.md is a concrete implementation of this idea.
- [[All Your Agents Are Going Async]] — HTTP is the wrong transport for agents that outlive connections. Sidekick's tmux + STATE.md architecture is an answer to "what transport, then?"
- [[Loop Engineering]] — Addy Osmani's meta-skill of designing systems that prompt agents. Sidekick's milestone reporting and stall watchdog are loop engineering primitives.
- [[Ralph]] — The Wiggum loop (`while :; do cat PROMPT.md | claude-code ; done`) is the simplest possible persistent worker. Sidekick is the industrial-grade version with crash recovery, stall detection, and team visibility.
- [[Bram]] — Jon Udell's hash-verified worklist lifecycle with PreToolUse hook enforcement across Claude Code and Codex CLI. Shares Sidekick's concern with making agent work verifiable and recoverable.
- [[Agent Orchestration]] — Hub page for multi-agent coordination patterns. Sidekick is a specific instance of the durable-worker pattern.

---
*Sources: [[raw/sidekick-skill]]*
*Last updated: 2026-07-18*
