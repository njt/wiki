# tmux for AI Agents (crystl)

crystl's guide to running AI coding agents in tmux is the most complete practical reference yet for the "tmux as agent harness" pattern — session-per-project naming, agent-tuned `.tmux.conf`, `send-keys`/`capture-pane` scripting, and a pane-scraping status line — and it is also, structurally, a vendor content-marketing piece: the closing "Where tmux stops" list is engineered to map one-to-one onto crystl's features. Both facts are useful. The working details are real and hard-won; the framing tells you exactly what the agent-aware terminal category thinks tmux lacks.

---

## What it argues

tmux is the right substrate for agents because of three properties: the tmux server owns the pty so agents survive disconnection, named sessions hold many agents in one window, and any shell can `send-keys` into or `capture-pane` out of a running agent — "no IPC to build, no API to wire up." The guide walks through `claude --worktree --tmux` (noting `--tmux` requires `--worktree` and uses iTerm2 native panes unless you pass `--tmux=classic`), then the hand-built path: one session per repo named for the repo, one window per task, panes only for the two or three you watch side by side.

## Key quotes

> Any shell, anywhere, can type into a running agent or read its screen. No IPC to build, no API to wire up. This is the property people build on, and it's why tmux keeps showing up in agent-orchestration scripts.

The whole reason tmux persists as agent infrastructure in one sentence: the terminal itself is the API. But it's an API to a *rendered frame*, not to agent state — which is exactly the crack everything later in the guide falls into.

> The match strings are whatever your agent version happens to print this week.

Unusual candour for a vendor guide. The status-line monitor works by string-matching a TUI's rendered frame ("esc to interrupt", "Do you want") on a five-second poll. The guide knows this is fragile and ships it anyway, because it's the best tmux can offer.

> An agent waiting on an approval prompt looks exactly like an agent thinking, which looks exactly like an agent that finished twenty minutes ago. All three are "a pane with some text in it."

The sharpest diagnosis in the piece, and the true motivation for the entire agent-aware-terminal category. The problem isn't multiplexing — it's *attention routing*: knowing which of six agents needs a decision from you at an unpredictable moment.

> Use it for that. When the thing you're managing is six agents that each need a decision from you at an unpredictable moment, the problem stopped being multiplexing.

The honest version of the pivot. tmux remains the right answer for remote boxes and fragile SSH links; the guide doesn't overclaim.

## Practical details worth keeping

- `send-keys` without `-l` interprets bare words as key names — a prompt containing "Enter" gets sent as a keypress; text and newline need separate calls, with a ~0.3s sleep for TUIs that debounce paste.
- `capture-pane -J` joins wrapped lines; without it "every grep you write against the output is wrong at the seams."
- `allow-rename off` stops agent TUIs renaming your carefully named windows; `escape-time 10` fixes mushy arrow keys inside interactive TUIs.
- Detached sessions render at the last client's size (or 80x24), so agents return to a mis-shaped pane until they redraw; `aggressive-resize` helps.
- `history-limit 100000` across a dozen panes is real per-pane memory, gone the moment the server dies. Nothing survives a reboot.

## Critical analysis

**The vendor shape is visible but not dishonest.** Every section ends in a capability tmux lacks, and every capability maps to a crystl feature ("Approvals surface instead of getting buried… No scraping, no polling"). That's textbook problem-solution marketing. What redeems it: the tmux material is genuinely good, the limitations listed are real, and the guide says plainly "keep tmux" for remote and long-running-server cases rather than claiming replacement.

**The scraping problem is structural, and the wiki has converging evidence.** This guide's status-line hack and candour about "what your agent version prints this week" are the same diagnosis made from the other side by sources that solved observability with state files rather than screen-scraping — [[How I run my agents across many accounts]] insists every agent's liveness be observable through state files *rather than* screen-scraping, and this guide inadvertently demonstrates why. Screen-scraping a rendered TUI is the terminal-era equivalent of scraping HTML: it works until the UI ships a change.

**It's the generic-tool counterpart to a real product category.** [[cmux]] is the same thesis (terminals designed for agent attention, not shell multiplexing) as a shipped Ghostty-based app with notification rings via OSC escape sequences; crystl is the pitch with structured turn/tool-call capture instead of scraping. This guide is valuable precisely because it documents the *baseline* — what running six agents in raw tmux actually costs — against which both products position themselves.

**Deeper theoretical backing exists.** [[The Day of a New Command-Line Interface Shell]] attacks the whole in-band-signalling terminal stack — control data and user data sharing one channel — which is the root cause of why `capture-pane` returns spinners, box-drawing and ANSI instead of what the agent said. This guide's "Where tmux stops" is that essay's argument, arrived at empirically from the agent side.

## Themes

#tool #pattern #concept

#tool — tmux as agent harness, and the agent-aware terminal (crystl, cmux) as the emerging replacement category.
#pattern — session-per-project naming, window-per-task, `send-keys`/`capture-pane` as the de-facto terminal API to agents.
#concept — attention routing over multiplexing: the scarce resource when agents run in parallel is knowing which one needs you.

---
*Sources: [[raw/tmux-for-ai-agents]], [[summary/tmux-for-ai-agents]]*
*Last updated: 2026-10-24*
