---
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---
# pi-observers

File-defined observer agents for the [pi](https://pi.dev) coding agent — a TypeScript extension that lets users declare background observers as Markdown files with YAML frontmatter. Observers watch one axis of quality each (memory, skills, goals, verification), propose short advisories, and a central reconciler decides what reaches the main agent. They are read-only, fire-and-forget, and never answer on the agent's behalf. Published as `pi-observers` on npm; requires pi >= 0.83.

---

## Architecture

Observers are defined in `.pi/observers/*.md` files with frontmatter fields for trigger (`on:`), visibility (`sees:`), tools (`tools:`), capabilities (`can: advise/veto`), delivery queue (`deliver:`), model, priority, and more. Three layers of precedence — project (trusted only) > user > builtin — all keyed on the `name:` field.

Each observer gets its own hermetically sealed `AgentSession`: no extensions, skills, prompt templates, or project settings leak in. Proposals arrive through pi's message queues the moment the observer's model call completes (arrival-driven delivery). A veto holds the run open; at most one veto per window.

The **Reconciler** arbitrates: deduplication by fingerprint, priority ranking, per-turn advisory budget (`maxAdvisoriesPerTurn`, capped at 10). Vetoes are governed by budget — per (observer, fingerprint) key, per-observer ceiling (2× vetoBudget), and session-wide ceiling (4× vetoBudget) — not by dedupe. Fingerprint encoding uses length-prefixed keys to prevent collision-based budget refunds. Deferred advisories queue (bounded at 100, oldest evicted first) for proposals over budget.

## Key Design Decisions

- **Arrival-driven, not drain-point:** Proposals reach the session the moment they're ready via pi's steer/followUp queues. No polling, no fixed drain intervals.
- **Non-blocking guarantee:** `bus.kick()` returns immediately; observer runs race against a timeout. The bus's slot releases regardless of run behavior.
- **Sanitized brand type:** TypeScript brand enforcing prompt injection defenses — only sanitized values enter assembled documents, and unforgeable section markers (a run of "=" one longer than the longest run in any rendered body) prevent attacker-controlled content from forging section boundaries.
- **Three-strike disable:** Three consecutive failures disable the observer for the session, with the last error shown in `/observers`.
- **Model resolution chain:** 6 steps — exact match → fuzzy (./- equivalence, date-stamp stripping) → any-provider → fallbacks in order → session model → disable. Auth-configured providers only considered.

## Bundled Observers

| Observer | Trigger | Capability | Default |
|---|---|---|---|
| `memory-recall` | turn_end | advise | on |
| `skill-recall` | before_agent_start | advise | on |
| `goal-tracker` | turn_end | advise, veto | on |
| `verification` | agent_settled | advise | off |

## Source

- **Repository:** [erans/pi-observers](https://github.com/erans/pi-observers)
- **Author:** Eran S. (erans on GitHub)
- **License:** MIT
- **Language:** TypeScript (~12,500 lines in src/ and test/)
- **npm:** `pi-observers`

---
*Fetched: 2026-08-07*
