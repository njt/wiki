# Xirp

Spotify's macOS desktop app for running parallel coding agents — Claude Code, Codex, or Gemini in persistent terminal sessions, each in its own Git worktree — with an optional hook into Spotify Portal for injecting Backstage Software Catalog and Workspace context through MCP. The open-source cousin of [[Orca]], [[Broomy]], and [[cmux]], distinguished by one thing: an enterprise context layer.

---

## Key Quotes

> "Run Claude Code, Codex, or Gemini in persistent terminal sessions and switch between them without losing state."

The category's table stakes, stated plainly. Persistent terminals + parallel sessions is no longer the differentiator — [[Orca]]'s daemon-persisted PTYs, [[Broomy]]'s side-by-side sessions, and [[cmux]]'s agent-aware panes all solved this in 2026. What Xirp is actually selling sits in the next quote.

> "Connect Spotify Portal to give agents access to Workspace wiki pages, catalog entities, resources, records, members, and prior sessions."

This is the whole product. Every open-source orchestrator manages *your* local context; Xirp manages *organizational* context. The Software Catalog and Workspaces are Backstage's knowledge substrate, and Xirp is the control plane that pipes it into a coding session over MCP. It's [[Organizational Intelligence Systems]]' "connect via MCP + skill files" recipe, productized by the company that invented Backstage.

> "Xirp does not replace your coding agent or source control provider. You continue to authenticate and configure each coding agent through its native CLI."

The honest positioning line, and the one that tells you what Xirp *isn't*: not a harness, not a runtime, not a repo. A thin control surface over tools you already own. It's the [[Smart Models Dumb Pipes]] instinct applied to the orchestration layer — but the flip side is that a thin convenience layer is exactly what commoditizes fastest ([[cmux]] names this risk directly).

> "A session launched from a catalog entity or Portal Workspace can receive relevant context through MCP."

The sharpest technical sentence in the doc. MCP is the delivery mechanism for organizational context — the same move [[Bringing MCP 2026-07-28 to Claude]] formalizes and [[Building Agents for Production Systems with MCP]] positions as the standard integration layer. Xirp is betting that "what does the agent already know about this service, this team, this past session?" is the question that separates enterprise agents from personal ones.

---

## Key Themes

- **#tool** — A desktop control surface for parallel coding agents. Same category as [[Orca]], [[Broomy]], [[Traycer]], [[cmux]], and [[Warp Agent CLI]]: one window, many agents, Git changes, files, rules, skills, and session status.

- **#pattern** — **Worktree-per-task isolation.** Each task gets its own Git worktree so agents don't touch the same checkout. This is now table stakes — [[Parallel Coding Agents Guide]] named it the foundational primitive, [[Orca]] and [[Agent of Empires]] implement it, and [[Introducing git-wt — Worktrees Simplified]] smooths its sharp edges. Xirp's contribution is making it a checkbox, not a git ritual.

- **#concept** — **Organizational context as the missing layer.** Portal holds context in the Software Catalog and Workspaces; Xirp injects it via MCP. This is the divide between "run my agents in parallel" (solved, crowded) and "run my agents with the company's memory" (the enterprise gap). The closest relatives: [[Agent Memory and Context]]'s taxonomy and [[Organizational Intelligence Systems]].

- **#pattern** — **Control plane, not harness.** Xirp explicitly declines to replace the coding agent or the source control provider. You keep authenticating through native CLIs. It's the thin-orchestrator position that [[Orca]]'s design notes also describe ("multi-agent orchestrator, not an agent itself").

---

## Critical Analysis

**The interesting part is what Spotify *isn't* rebuilding.** By mid-2026 the "desktop app that runs parallel agents in worktrees" category is crowded — [[Orca]] (open source, 20+ agents, SSH, mobile), [[Broomy]] (MIT, side-by-side + IDE), [[cmux]] (native macOS, attention rings), [[Traycer]] (17+ agents, real-time collaboration). Xirp's terminal-session and worktree features are commodity. Its bet is that the durable value isn't orchestration mechanics but the **Portal context layer** — the thing only Spotify can ship, because it owns Backstage and the Software Catalog sits at the center of the orgs that adopted it.

**That's also its ceiling.** The Portal features are what make Xirp more than [[Orca]] — but they only matter to teams already running Backstage with a populated Catalog and Workspaces. For everyone else, Xirp is a macOS-only beta that runs three agents in worktrees, which they can already do for free. It's a wedge product for the Backstage install base, not a general-purpose competitor.

**The beta scope is honest in a way that cuts both ways.** macOS-only, three agents, and *manual* session upload are all rough edges — but "manual session upload" is the tell. The promise of "upload a completed Workspace session for teammates and future agents" is the [[Agent Memory and Context]] dream of persistent organizational memory, and doing it manually means the flywheel isn't spinning yet. If automatic session upload lands, Xirp becomes a genuine memory sink; until then it's a convenience.

**"Rules and skills" in the control surface is a convergence signal.** That Xirp manages *rules and skills* from the same app that manages terminals and Git is the same realization as [[Loop Engineering]] and [[Steering Claude Code]]: the durable artifacts of agentic work are configuration and context, not the session itself. A desktop app that treats CLAUDE.md-style rules as first-class managed objects is the natural next step after everyone learned the harness matters more than the model.

**The honest "doesn't replace your agent" line is right, and risky.** It correctly refuses to be a harness. But a thin control surface is precisely the layer that gets absorbed — [[cmux]] already predicts notification rings will be copied into iTerm2/Warp/Ghostty, and the same logic applies here. Xirp's defense is the Portal MCP integration, and that defense is real only inside the Backstage ecosystem.

---

## Related Pages

- [[Orca]] — The closest open-source analogue: parallel worktrees, persistent terminals, desktop control surface. Xirp is Orca minus the 20-agent/SSH/mobile ambition, plus organizational context
- [[Parallel Coding Agents Guide]] — The theory Xirp mechanizes: worktree-per-agent isolation and the orchestrator maturity tiers. Xirp is tier-3 "dedicated orchestrator," and its Portal layer answers the guide's unaddressed merge-conflict/coordination gap
- [[Broomy]] — Same category, opposite philosophy: agent-agnostic local-first vs. catalog-backed enterprise
- [[cmux]] — The attention/control-room problem Xirp's "one control surface" also targets, from the native-macOS-terminal direction
- [[Introducing git-wt — Worktrees Simplified]] — The worktree primitive Xirp turns into a checkbox
- [[Organizational Intelligence Systems]] — The MCP + context-grounding recipe Xirp productizes on Backstage's substrate
- [[Agent Memory and Context]] — What Portal's "prior sessions" upload is reaching toward: persistent organizational memory

---
*Sources: [[raw/xirp]], [[summary/xirp]]*
*Last updated: 2026-08-14*
