# Domenic Denicola's Agentic Coding Setup

Domenic Denicola — former Chrome engineer and WHATWG spec editor — documents his personal agentic coding infrastructure as of July 2026: a disposable Ubuntu VM on a home server, Tailscale for private networking, Claude Code and Codex running in YOLO mode, git worktrees for parallelism, chezmoi for dotfile sync, and the ChatGPT desktop app as the surprise-winning thin client. The setup delivers phone-based development "almost for free," letting him fix bugs from a train and merge PRs before reaching his stop.

---

## Key Quotes

> "The real winner is the ChatGPT app over SSH. It's truly excellent as a thin client. It handles disconnects and backgrounding very gracefully—a critical feature that many remote development tools get wrong."

This is the article's most actionable claim. Denicola has tried both major agent harnesses extensively, and his verdict is clear: ChatGPT's SSH client is better at being a thin client than Claude's desktop app. The specific failure modes he names — session death on client close, no filesystem-based session grouping, broken filesystem explorer — are concrete and falsifiable. If Anthropic wants to win the remote-development use case, this paragraph is a ready-made bug report.

> "I configure both Claude Code and Codex CLI so they don't ask for any approvals, and my user is in the sudoers file so they can access whatever they need. People have valid concerns [...] I haven't found that to be an issue in practice—and I run a lot of agents!"

The honest YOLO admission. Denicola acknowledges the risks (deleting repos, leaking secrets, escaping the VM) and then waves them off based on empirical experience rather than theoretical safety guarantees. This is the same pragmatic posture as [[How Boris Uses Claude Code]], where Boris describes pre-allowing safe commands rather than going full `--dangerously-skip-permissions`. Denicola goes further — full sudo access — and reports no problems. The difference may be that Denicola's disposable VM provides blast-radius containment that Boris's local setup doesn't.

> "I dislike how sessions are strongly tied to the folder path on the VM (renaming a project causes all your session history to disappear)."

A sharp observation about a hidden brittleness in current agent infrastructure. Session history is keyed to filesystem paths, not project identity. This is a design choice nobody made consciously — it's just the path of least resistance in implementation — and it has real consequences for anyone who reorganizes their projects. The fix (keying sessions to a project UUID or git remote) is obvious; nobody has shipped it.

> "I'm sure in six months this entire post will be quaintly obsolete. What a time to be alive!"

The closing line is disarming and accurate. Articles like this have a half-life measured in months, which is exactly why they're valuable: they capture what worked in a specific moment before the ground shifts again.

## Key Themes

#agentic-coding #setup #claude-code #codex #worktrees #tailscale #vm #personal-workflow #yolo #remote-development #mobile-development

### The Disposable VM Pattern

Denicola's setup is a specific flavor of the VM-based approach: a single Ubuntu Server VM on Hyper-V, quick to create and quick to replace. This isn't the cloud-VM-per-task model (contrast [[Claude Code on the Go]], which uses cloud VMs for parallelism). It's a personal server that happens to be virtualized, running on always-on home hardware. The disposability is a safety property — if an agent trashes the VM, you rebuild it — but it's not the primary motivation.

### Tailscale as Agent Infrastructure

Tailscale does heavy lifting in this setup: SSH with auto-accept rules (no key management), HTTPS certificates for secure-context dev servers, Taildrive for file sharing, and VPN for phone access. This isn't just networking — it's the connective tissue that makes a multi-device agent workflow feel like a single machine. [[Agentcookie]] takes this one step further, using Tailscale to sync browser cookies and API tokens to an agent machine, eliminating per-site auth ceremony.

### The ChatGPT App Wins as Thin Client

Denicola's comparison of ChatGPT vs. Claude desktop apps is the most detailed public head-to-head I've seen:

| Feature | ChatGPT | Claude Desktop |
|---------|---------|----------------|
| SSH support | Excellent | Poor (kills sessions on close) |
| Disconnect handling | Graceful | Sessions die |
| Session grouping | Filesystem-based | None |
| Worktree integration | Button → VS Code over SSH | Broken (path issues) |
| Filesystem explorer | Works | Requires repeated refresh |

The Claude desktop app was designed for local use and retrofitted for remote; ChatGPT was apparently designed with remote as a first-class scenario. This matters more than model quality for Denicola's workflow — he runs both Claude Code and Codex CLI on the VM, and the thin client is just the access mechanism.

### Worktrees as Parallelism Primitive

Git worktrees solve the "parallel workstreams without interference" requirement. Each agent gets its own directory and branch. But Denicola surfaces a concrete Claude Code bug: worktrees are created inside the project directory (`.claude/worktrees/`), which causes `node_modules` resolution issues and can trigger recursive `claude` command discovery. The workaround is to place worktrees outside the project tree — a one-line config change that isn't the default.

This converges with [[Introducing git-wt — Worktrees Simplified]] and [[Fable Open-Sourced NanoClaw's PR Factory]], both of which use worktrees as the isolation primitive for parallel agent sessions.

### `tportless`: Dev Servers on the Tailnet

Denicola's `tportless` wrapper around [[Portless]] solves a genuine coordination problem: each agent's dev server needs a unique port, HTTPS, and a stable URL reachable from any device on the tailnet. The solution (`https://agents-base.tail234567.ts.net:8443/`) gives every agent a secure context for Service Worker and Web API testing, with port allocation that prevents collisions between parallel agents. This is the kind of infrastructure that feels obvious after you hear it but takes real engineering to get right.

### Phone-Based Development Falls Out Free

The mobile workflow is the most impressive emergent property: notice a bug → open ChatGPT on phone (Tailscale VPN auto-connects) → ask agent to fix it → get push notification with preview URL → confirm and merge PR. This isn't a separate mobile feature — it's the exact same infrastructure serving a different form factor. [[Claude Code on the Go]] described a similar pattern but positioned it as the main use case; Denicola treats it as a happy side effect of having designed the system for remote access.

### Chezmoi for Agent Config Sync

Using chezmoi (a dotfile manager) to sync AGENTS.md, skills files, and preferences across machines is a practical insight. Most practitioners copy-paste these files or maintain them per-machine. Denicola treats agent configuration as dotfiles — version-controlled, machine-synced, and set up by GPT. This is the smallest change with the largest quality-of-life impact in the whole article.

### The Gaps: Dev Containers, Session Portability, Transcript Backup

Three gaps Denicola names:

1. **Dev containers** would provide better isolation than VMs. This is the same observation driving [[Nango — Running Untrusted Customer Code at Scale]] and the broader sandboxing literature in [[Security and Sandboxing]].

2. **Session-to-path coupling** is a design bug. Sessions should key to project identity, not directory paths. This is a tractable engineering problem that nobody has prioritized.

3. **Transcript backup** to a durable store (like a private GitHub repo) for "nostalgia and design record-keeping." [[engineering-notebook]] and [[session-analysis]] address pieces of this, but Denicola wants something simpler: just archive everything, search later. The value isn't debugging — it's remembering how you arrived at a design.

## Critical Analysis

**The setup is elegant because it's boring.** Every component is off-the-shelf: Ubuntu Server, Tailscale, git worktrees, chezmoi, Portless. Denicola didn't build a custom orchestration layer or a novel agent framework. He wired together existing tools and let the agents operate within that infrastructure. This is the opposite of [[Fleet Supervisor (sermakarevich)]] or [[Ruflo]], which build custom orchestration layers. Denicola's approach wins on simplicity and loses on features (no queue management, no agent-to-agent communication, no structured state tracking). For a solo developer, that's the right trade.

**The YOLO admission is significant because of who's making it.** Denicola is not a reckless hobbyist. He co-edited the Streams, URL, and Console standards at WHATWG; he worked on Chrome at Google; he co-authored the HTML spec. When someone with that background says full-sudo agents haven't caused problems in practice, it carries different weight than the same claim from a vibe coder. The caveat is that he's running on a disposable VM — the blast radius is bounded even if the permissions are not.

**The ChatGPT-vs-Claude thin-client comparison is actionable and should embarrass Anthropic.** Denicola's critique of Claude Desktop (sessions die on close, no filesystem grouping, broken explorer) reads like a product manager's priority list for the next sprint. These aren't fundamental architectural problems — they're features that weren't prioritized because the team assumed local-first usage. Meanwhile, OpenAI shipped a better remote development experience, and Denicola — who runs Claude Code on the VM — uses ChatGPT's app to access it. That's a distribution vulnerability for Anthropic.

**The phone workflow reveals what "agentic coding" actually means.** It's not about typing less code. It's about the collapse of place-dependence. The developer doesn't need to be at a desk, or even awake. The agent is a persistent service you occasionally check on, not a tool you wield. [[Automating Myself Out of Development]] describes the same shift from the automation angle; Denicola describes it from the UX angle. Both converge on the same insight: the interface is becoming a notification, not a terminal.

**The six-month obsolescence prediction is the most honest thing in the article.** Agentic coding infrastructure is in the frothy phase where every month brings a better way to do something. Writing down what works today isn't about creating a permanent reference — it's about capturing a waypoint so you can measure how far you've traveled. This article will be quaintly obsolete. That's the point.

**One direction the infrastructure is evolving is toward fully integrated desktop orchestrators.** [[Orca]] productizes the exact pattern Denicola describes — parallel git worktrees, SSH remote execution, mobile monitoring — into a single Electron app with daemon-persisted terminals that survive crashes and a hook-based agent status system that eliminates the "is Claude still working?" question. It's the route where you buy the platform rather than assembling it from VMs and Tailscale.

---

*Sources: [[raw/domenic-agentic-coding-setup]]*
*Last updated: 2026-07-25*
