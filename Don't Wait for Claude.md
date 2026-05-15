# Don't Wait for Claude

Jay McCarthy's argument that the bottleneck in AI-assisted development isn't model speed — it's your ability to manage parallel sessions without losing context. His solution: externalize state so resuming a session costs zero mental recall, and his tool `jc` implements this as a three-pane macOS app that cycles through problems by priority rather than making you hunt through tabs. The thesis: going from four cycles an hour to twelve isn't about Claude getting faster; it's about you getting better at the human side of the loop.

---

## Key Quotes

> "The problem isn't that you can't *run* multiple sessions. It's that you can't *manage* them."

This is the essay distilled to one sentence. Every tool that spawns agents hits this wall — the orchestration layer is the hard part, not the runtime. See [[Agent Orchestration]], [[Agent of Empires]].

> "Coming back is the hard part."

McCarthy names the specific cognitive tax: not the context switch itself, but the *re-entry* cost. This is the same insight that drives [[Agent Memory and Context]] and [[napkin]] — the scratchpad exists so you don't have to reconstruct state from memory.

> "This isn't extra work. It's the review work you should be doing anyway, just done at the right time."

The reframe that makes the system self-sustaining. Notes written during review become the next cycle's instructions. By the time you're ready to send, "the instruction writes itself." This is [[Compound Engineering]] applied to the human side: build a system, not a habit.

> "Claude finishes and nothing happens."

McCarthy identifies notification failure as a first-class problem. No badge, no sound, no indicator. You discover sessions finished minutes ago, or you waste time checking tabs that don't need attention. [[Claude Lamp]] solves this with ambient light; McCarthy solves it with a priority-ordered problem queue.

> "The practice is sound. The manual implementation leaks at every joint."

An honest engineering admission. The idea (externalize state, review-as-you-go, problem-priority navigation) is correct, but the friction of manual tooling kills it. Notes take four actions to write one line. Navigation loses the thread. The tool matters as much as the workflow.

> "If you have improvements, have your Claude open a PR against mine. I don't accept human-authored code."

The closing provocation. Not just "use AI to contribute" but "I won't even look at code a human typed." A statement about where McCarthy draws his own [[Agent Coding Workflow]] boundary.

---

## Key Themes

- **#pattern** — Externalized state as the solution to context-switching cost. Notes aren't documentation; they're the next prompt.
- **#pattern** — Problem-priority navigation over session-priority. Don't pick a session; pick the next thing that needs human attention (permits > unreviewed > unsent).
- **#tool** — `jc` as a three-pane layout (terminal, TODO, diff) with a single keybinding for note capture and a priority queue for what to do next.
- **#concept** — "Review work you should be doing anyway, just done at the right time." The notes you write during review ARE your next instructions. No double work.
- **#person** — Jay McCarthy, creator of `jc`. Builds tools for multi-session agent management on macOS.

---

## Critical Analysis

McCarthy has identified a real and under-discussed bottleneck. The industry is obsessed with model speed and agent autonomy, but the human's attention budget is the actual constraint on throughput. His diagnosis is sharp: the problem isn't spawning sessions, it's *resuming* them.

The three failure modes he identifies in the DIY implementation are painfully accurate. Notes friction is real — the difference between "think of a thing and type it" and "navigate to the right pane, find the right heading, scroll past history, then type" is the difference between a workflow and a chore. Notification failure is an indictment of terminal-based tooling: Claude Code is invisible when it finishes unless you're watching. Navigation failure compounds both: when you can't tell at a glance which tab needs you, you click through blindly and lose context each time.

His solution — a priority queue of problems, not sessions — is clever. It treats human attention as the scarce resource and optimizes for it. Permission prompts first (they block Claude), then unreviewed diffs (the work product), then unsent notes (the next instructions). This is the right ordering.

Where I'm skeptical: the `jc` tool is macOS-only, editor-locked, and early-stage (it's a personal project, not a product). The three-pane layout is opinionated — it works if your mental model matches McCarthy's, but might not if you think about sessions differently. And the workflow assumes you're disciplined enough to write notes during review; if you skip that step, the system collapses into ordinary tab-switching. The tool reduces friction but doesn't eliminate the discipline requirement.

The "I don't accept human-authored code" closer is a flex, not a principle. It's funny and provocative, but it's also a statement about the kind of development McCarthy is optimizing for — one where human code review is the bottleneck, and agent-authored PRs are the unit of work. That's not everyone's workflow yet, but it's where the field is heading.

Compare with [[How Boris Uses Claude Code]] (the workflow McCarthy explicitly references), [[Agent of Empires]] (similar multi-session management via tmux and git worktrees), [[Collaborator]] (spatial arrangement as an alternative to tab-switching), and [[Managing Agents via Kanban Boards]] (board columns as the problem-priority queue). McCarthy's contribution is the tightest articulation of *why* these tools are necessary: because resuming a session is a retrieval problem, and human memory is the worst retrieval system available.

---

*Sources: [[raw/jc-workflow]]*
*Last updated: 2026-05-15*
