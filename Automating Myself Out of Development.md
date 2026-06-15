# Automating Myself Out of Development

Nune Isabekyan's field report on progressively removing herself from the development loop: from interactive Claude Code sessions through a cron-driven daemon that implements features overnight, using GitHub issues as a kanban board. A rare, honest account that tracks the *human experience* of delegation — not just the technical architecture — and lands on a checkpoint-style workflow where Claude works while she sleeps and she reviews over morning coffee.

---

## Key quotes

> "The actual *intelligence* of the implementation is inside the Claude subprocess."

The daemon script (`tick.sh`) is deliberately boring — a cron-fired bash script that locks, finds the oldest `ready` issue, and spawns `claude -p`. The shell is just scaffolding; all the smarts live in the LLM call. This is the same insight as [[Smart Models Dumb Pipes]] applied to personal tooling.

> "I wanted *checkpoint-style* communication — Claude does a chunk, leaves a clear artifact and a question, and I come back the next morning."

The shift from synchronous to asynchronous collaboration. Not "vibe coding" — more like managing a remote contractor who works your timezone's night shift and leaves a daily standup note.

> "The brainstorm log earned trust because I can scan it in minutes and see exactly where the model was guessing."

The auto-brainstorming step produces a forensic receipt: per-section confidence levels with sources. This is the transparency mechanism that makes progressive delegation possible — you don't trust the output, you trust your ability to audit it quickly.

> "The pipeline doesn't remove the need for static analysis, code review, architecture review, rework, or security audits — it makes those *more* important because mediocre code can land faster."

The throughput/quality inversion. Higher velocity means the guardrails aren't less necessary — they're *more* critical. This echoes the [[Guardrails and Feedback Loops]] thesis: deterministic enforcement becomes load-bearing when generation speed increases.

> "If anyone tells you with 100% confidence how AI must be used in your development process or organisation, run. They haven't tried it themselves."

The article's closing line. A hedge against the certainty merchants — itself a form of intellectual honesty that this entire wiki tries to model.

---

## The architecture

The system is a **GitHub issues state machine driven by a cron daemon**:

1. **Backlog repo** — GitHub issues with labels as state (`needs-enrichment` → `enrichment:needs-review` → `ready` → `in-progress` → `needs-attention` / `branches-ready`)
2. **`tick.sh`** — runs every 15 min on EC2. Takes a file lock, refreshes `gh` token, pulls backlog, resets stuck issues, claims oldest `ready` issue, spawns `claude -p`
3. **`/feature-gh` skill** — Claude Code skill that knows how to traverse the issue lifecycle: brainstorm, spec review, plan creation, plan review, implementation, merge. Each phase is an isolated subagent with its own context window
4. **Artifacts** — specs and plans live in `specs/issue-N/` directories; state tracked in `state.json` so runs can resume after failure
5. **Human gates** — five explicit touch points, each requiring a label flip to proceed

The key architectural insight: **each phase gets a clean context window**. The brainstorm subagent never sees implementation noise; the reviewer never sees brainstorming ramble. This solves the context pollution problem that plagued Phase 0's multi-tab approach.

Compare to [[StrongDM Factory Techniques]] (similar "dark factory" concept but focused on code-as-opaque-weights) and [[Serf]] (non-interactive coding agent but without the orchestration layer). The GitHub-issues-as-board pattern converges with the [[Agent Orchestration]] kanban observation — planner/worker/judge mapped onto issue labels.

---

## Critical analysis

**The real contribution is the phased narrative, not the architecture.** Plenty of people have built cron+LLM pipelines. What's valuable here is the recorded deterioration of the interactive paradigm: Phase 0 (multi-tab context fatigue) → Phase 1 (EC2 isolation, dumber Claude without context) → Phase 2 (checkpoint-style communication) → the enrichment/auto-brainstorm layers that keep getting added as each bottleneck reveals the next. It's a story about *the problems you discover only by living in the workflow*, not the ones you anticipate in design.

**The "brainstorm log" as an audit receipt is underrated.** Most auto-spec approaches give you a spec; Isabekyan's gives you a spec *and* a confidence-annotated Q&A trail. This is the difference between "trust the model" and "trust your ability to verify the model." It's a [[Guardrails and Feedback Loops]] pattern: don't prevent errors, make them visible.

**The bottleneck shift is the most honest observation in the piece.** "I don't have time to write code" became "I don't have time to brainstorm and review thoroughly enough." Delegation doesn't eliminate work — it moves it higher in the stack. This is consistent with the [[Agent Coding Workflow]] maturity spectrum: the more you delegate implementation, the more design and review become your full-time job.

**The EC2 migration exposing context leaks is a warning.** Her CLAUDE.md and memory files were "too inter-connected and messy" — context had bled between projects in ways invisible on a single machine. This is the [[Agent Memory and Context]] problem in microcosm: context isn't just what you give the model, it's what the environment implicitly provides.

**Skepticism about fully autonomous brainstorming is correct.** The auto-brainstorm is opt-in, produces auditable receipts, and still has a human gate. The warning about entering "broken telephone" territory with self-suggesting agents is well-placed — each delegation step adds noise, and recursive delegation compounds the error.

**The missing piece: cost accounting.** Isabekyan admits she doesn't know if this is actually faster or cheaper than doing it herself. Without token cost tracking and time-to-merge metrics, the whole edifice could be elaborate procrastination disguised as automation. The [[Agent Flywheel]] requires measurement to close the loop.

---

## Themes

#workflow #orchestration #claude-code #solo-dev #github-issues #kanban #daemon #context-management #delegation #cron #checkpoint-workflow #auditability

---

## Related pages

- [[Agent Coding Workflow]] — The maturity spectrum this workflow sits on
- [[Agent Orchestration]] — Kanban boards as the human-agent interface
- [[Addy Osmani's Workflow]] — Another solo dev's AI-assisted workflow
- [[Claude Code Mastery]] — CLAUDE.md as compounding infrastructure
- [[Agent Memory and Context]] — The context pollution problems encountered in Phase 1
- [[Guardrails and Feedback Loops]] — Gates and review as velocity enablers, not blockers
- [[StrongDM Factory Techniques]] — Similar "dark factory" patterns, different emphasis
- [[Serf]] — Non-interactive coding agent, but without the orchestration
- [[Specifications as the Product]] — Spec-driven development where code is disposable
- [[From AI Studio to AI Forge]] — "Human changes altitude" framing for supervisory control
- [[Moltbook]] — Heartbeat-driven agents; the lethal trifecta in production
- [[Agent Flywheel]] — The measurement/feedback loop this workflow still needs
- [[Smart Models Dumb Pipes]] — The daemon is dumb; the LLM subprocess is smart
- [[Agent Orchestration for the Timid]] — Similar gradual-delegation approach

---

*Source: [Automating Myself Out of Development](https://www.thoughtfultechnologist.com/p/automating-myself-out-of-development) by Nune Isabekyan, April 28, 2026. Ingested 2026-06-15.*
