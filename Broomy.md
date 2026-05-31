# Broomy

A free, MIT-licensed Electron desktop app (TypeScript + React) that runs multiple terminal-based AI coding agents side-by-side in a single window, with built-in IDE features (file editing, git, code review) and session notifications. Broomy is the answer to "which terminal tab is Claude Code in?" — agent-agnostic, local-only, no accounts, no telemetry.

---

## Key Quotes

> "Terminal tabs everywhere. Which agent finished? Which one needs input? What branch is it on? What files did it change?"

This is the pain point that every multi-agent user recognizes. The landing page copy is doing real diagnostic work here — naming the specific chaos rather than vague "agent management" language.

> "Not about rubber-stamping AI output"

The code review feature positions itself as human-in-the-loop quality assurance, not automated approval. This is the right posture. Whether the implementation delivers on that promise is the open question at Public Preview stage.

> "You're not forced to hand everything to the AI."

The built-in IDE (file editing, git staging, committing) means Broomy isn't a YOLO-mode dashboard. You can review, edit, and commit manually without leaving the window. This is a deliberate design choice that pushes against the [[The Dark Factory is a DOT File]] maximalist position.

> "No telemetry. No paid tier. No 'open core' bait-and-switch."

Rare and refreshing. In a space filling up with "free for now" tools, this is a credible commitment. MIT license means fork-and-own is always an option.

---

## Key Themes

#tool #coding-agent #multi-agent #desktop-app #local-first #code-review #open-source

---

## How It Works

Broomy wraps terminal-based coding agents (Claude Code, Aider, Codex, Gemini CLI — anything that runs in a terminal) in an Electron shell with three integrated capabilities:

1. **Session dashboard** — See all active agents, their status (working/idle/needs-input/finished), and desktop notifications when tasks complete so nothing slips through. Each agent gets a dedicated terminal session. The pitch: "Run Claude Code on your backend, Aider on your frontend, and keep a terminal open for your docs — all in one window."

2. **AI-guided code review** — Summarizes diffs, flags potential issues, links to changes. The stated goal is keeping humans in the loop, not automating approval.

3. **Built-in IDE** — File tree, editor, git status/staging/committing. The pitch is that you don't need to switch to VS Code or terminal to review and commit agent output.

Tech stack: TypeScript + React + Electron, pnpm + Node.js. Currently Mac-only; Windows/Linux "coming soon." Status: Public Preview ("stable enough for daily use, but you may run into issues").

Explicitly supports Claude Code, Codex, Gemini CLI, and any terminal-based agent. Adding support for new agents is described as trivial — Broomy treats agents as opaque terminal processes.

---

## What's Interesting

**The all-in-one bet.** Unlike [[Dorothy]] (MCP-first orchestrator with Kanban and automations) or [[Agent of Empires]] (tmux + worktree session manager), Broomy bundles session management, code review, and a basic IDE into one window. This is either integration genius or feature bloat — the Public Preview label suggests they haven't proven which yet.

**The IDE inclusion is the differentiator.** Dorothy gives you agent orchestration. Collaborator gives you spatial arrangement. Broomy gives you a code editor and git interface. If you actually need to read and modify agent output (and you should — see [[Slowing the Fuck Down]]), having the IDE colocated with the agent sessions reduces context-switching friction. The question is whether an embedded editor is good enough to replace the tools developers already use.

**Agent-agnostic design is the right move.** Unlike tools that hardcode Claude Code or Codex integration, Broomy treats agents as opaque terminal processes. This means it can't offer deep integration (no structured output parsing, no tool-use visibility), but it also means it works with everything and won't break when agent CLIs change their output format. It's the [[Smart Models Dumb Pipes]] pattern applied to the agent management layer.

**Local-only with no accounts is a trust play.** In a space where [[PiClaw]] requires Docker, [[Claude Sidecar]] needs API keys, and many tools phone home, Broomy's "nothing leaves your machine" stance is a genuine differentiator. Combined with MIT licensing, this makes it the most forkable multi-agent UI.

---

## Critical Analysis

**What's good:**

The pain point diagnosis is precise. Anyone running multiple agent sessions knows the tab-hunting problem. Broomy identifies it cleanly and proposes a contained solution — it doesn't try to be a platform, doesn't add MCP servers, doesn't introduce a task queue. Just windows, terminals, and a code editor. The desktop notification for completed tasks is a small but high-leverage feature — the difference between "I wonder if Claude finished" and being pulled back at the right moment.

The code review feature, if implemented well, addresses a real gap. Most multi-agent tools focus on *running* agents; few help you *review* their output. The "not rubber-stamping" language suggests they understand the difference between review and approval.

MIT license + no accounts + no telemetry is the strongest open-source commitment in the multi-agent UI space. Compare to tools that are "open source" but have paid tiers or cloud dependencies. This is closer to the [[Zed]] philosophy than the SaaS-graduation playbook.

**What's concerning:**

Mac-only at this stage of the Electron ecosystem is a choice. Electron runs everywhere — shipping Mac-only in Public Preview suggests either a small team prioritizing their own platform, or Mac-specific features (like native git integration) that don't port cleanly. Either way, the "coming soon" promise for Windows/Linux should be discounted until there's a release date.

The embedded IDE is the riskiest feature. VS Code, Zed, and terminal-based editors aren't going to be replaced by an Electron app's code pane. If the editor is just "good enough for quick fixes," it's valuable. If it's positioned as a replacement, it's hubris. The landing page is careful here — "you're not forced to hand everything to the AI" — but the feature list oversells.

No structured agent integration means Broomy can't show you what tools an agent is using, what files it's reading, or whether it's stuck in a loop. It sees terminal output, not agent state. For power users, this is a meaningful limitation — compare to [[Dorothy]]'s MCP-driven agent lifecycle management or [[AgentsView]]'s session-aware analytics. Broomy's simplicity is also its ceiling.

**The real comparison:**

Broomy sits in a crowded space where the competition isn't other tools — it's tmux. A developer who's comfortable with tmux can already run multiple agents side-by-side, see their output, and switch between them. Broomy's value-add is the IDE and the code review feature. If those aren't good enough to replace your existing editor + review workflow, Broomy is just tmux with more RAM usage.

The tool it most resembles is [[Collaborator]] — both are Electron desktop apps that arrange coding work spatially. Collaborator bets on infinite canvas; Broomy bets on integrated editing + review. Different philosophies, same technical substrate.

---

## Cross-References

- [[Dorothy]] — The most direct comparison: MCP-first multi-agent orchestrator with Kanban and automations. Broomy is simpler, Dorothy is more powerful. Different answers to the same question
- [[Agent of Empires]] — tmux + git worktree session manager. The infrastructure-layer approach; Broomy adds UI where AoE adds plumbing
- [[Collaborator]] — Infinite canvas desktop app for agents and terminals. Same Electron substrate, different spatial metaphor
- [[Claude Sidecar]] — Parallel AI windows with structured fold-back. Broomy's multi-agent view is a different approach to the same concurrency problem
- [[Claude Chic]] — Alternative Claude Code TUI. Like Broomy, bets that the default interface isn't good enough
- [[AgentsView]] — Analytics for agent sessions. Complementary: Broomy runs agents, AgentsView analyzes them
- [[Parallel Coding Agents Guide]] — The practice guide for what Broomy mechanizes
- [[Agent Orchestration]] — The theoretical patterns behind multi-agent coordination
- [[acpx]] — Headless CLI for 16+ coding agents. The terminal-native alternative to Broomy's GUI approach
- [[PiClaw]] — Self-hosted AI workspace. Similar "everything in one place" ambition, containerized instead of Electron
- [[What I learned building an opinionated and minimal coding agent]] — The "four tools, not forty" counterpoint to Broomy's feature bundling
- [[Smart Models Dumb Pipes]] — The architectural pattern Broomy's agent-agnostic design follows
- [[Slowing the Fuck Down]] — The human-in-the-loop philosophy Broomy's code review feature claims to enable
- [[Zed]] — Another open-source native editor betting that the default tools aren't good enough. Different domain, same spirit
- [[MobileVibe]] — Mobile agent control. Complementary: Broomy on desktop, MobileVibe on phone

---
*Sources: [[raw/broomy]]*
*Last updated: 2026-05-31 (re-ingested with surf browser content)*
