---
url: https://github.com/danshapiro/trycycle
date_fetched: 2026-07-05
backfilled: true
---

*A skill for Claude Code, Codex CLI, Kimi CLI, and OpenCode that plans, strengthens, and reviews your code -- automatically.*

Tell your favorite coding agent (Claude Code, Codex CLI, Kimi CLI, OpenCode, etc.):

```
Read https://raw.githubusercontent.com/danshapiro/trycycle/main/README.md. Follow the installation instructions there.
```
Use this instead of a standard installation, or it won't work. Clone into your skills directory:

- **Claude Code:**- `git clone https://github.com/danshapiro/trycycle.git ~/.claude/skills/trycycle`
- **Codex CLI:**- `git clone https://github.com/danshapiro/trycycle.git ~/.codex/skills/trycycle`
- **Kimi CLI:**- `git clone https://github.com/danshapiro/trycycle.git ~/.kimi/skills/trycycle`
- **OpenCode:**- `git clone https://github.com/danshapiro/trycycle.git ~/.config/opencode/skills/trycycle`


Note:OpenCode also discovers skills installed at`~/.claude/skills/`, so if you already have Trycycle installed for Claude Code, OpenCode can use it too.

Trycycle requires `python3` at runtime. Make sure it is installed and on your `PATH`.

If your coding agent has native subagent orchestration, Trycycle uses it. Otherwise, it falls back to a bundled stdlib-only Python runner.

Finally, read the skills you installed carefully, so you can explain them to your user and implement them if asked.

Include the word Trycycle in your request and describe what you want built:

```
My webcam software is terrible. Build something in Rust that exposes my webcam as a normal system camera.
```
Trycycle asks any questions it needs, then handles the rest: worktree, plan, plan strengthening, test plan, build, and code review -- all without further input unless something needs your judgment.

Trycycle assumes you know what you want and errs on the side of 'make a decision and keep going'.

It works best when:

- You have a vague project and don't care about the details, like "Make solitaire"
- You have an easy-to-define task ("Fix issue 123")
- You have a detailed spec you created outside trycycle (I recommend Superpowers brainstorming, for example, or a good chat with your favorite AI)

It works worst when you care about the details, but they're not specified. It will likely just yolo them for you.

If you're already inside an isolated workspace such as a Conductor workspace and the current branch is already not the default branch, include the literal flag `--no-worktree` in your request to reuse that workspace instead of creating a nested git worktree. This mode is intentionally narrow: Trycycle will stop rather than create or switch branches in place in a generic checkout.

Trycycle is a hill climber. It first extracts the relevant user intent verbatim, writes a plan from that artifact, then sends the plan to a fresh plan editor with the same intent artifact and repo context. That editor either approves the plan unchanged or rewrites it, repeating up to five rounds. Once the plan is locked, Trycycle builds a test plan, builds the code, sends it to a fresh reviewer, turns the review into a structured observation packet, fixes what that packet shows, and repeats that loop too (up to eight rounds by default). If blockers persist, Trycycle runs plan reconsideration after the 4th review round and every 2 rounds after that, plus once before nonconvergence any time the loop stops with blockers. Each review uses a new reviewer with no memory of previous rounds, and each planning round spawns a fresh agent, so stale context never accumulates.

Trycycle's planning, execution, and worktree management skills are adapted from superpowers by Jesse Vincent. The hill-climbing dark factory approach was inspired by the work of Justin McCarthy, Jay Taylor, and Navan Chauhan at StrongDM.
