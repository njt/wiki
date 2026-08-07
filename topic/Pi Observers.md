# Pi Observers

Pi Observers is a TypeScript extension for the Pi coding agent that introduces file-defined background observer agents. Each observer watches one axis of quality, proposes short advisories, and a central reconciler decides what reaches the main agent. The system is a worked example of several emerging agent architecture patterns: fire-and-forget background monitoring, hermetically sealed sub-sessions, arrival-driven message delivery, and adversarial budget defense against model-chosen identifiers.

---

## Architecture

### Observer Lifecycle

Observers are Markdown files in `.pi/observers/` with YAML frontmatter. Each declares: when it wakes (`on:`), what context it sees (`sees:`), which read-only tools it may use (`tools:`), whether it advises or vetoes (`can:`), which delivery queue its proposals ride (`deliver:`), and its model preferences. Four observers ship bundled: `memory-recall` (finds relevant notes), `skill-recall` (suggests skills to load), `goal-tracker` (enforces declared goals, may veto), and `verification` (checks claimed work against tool records).

Three layers of precedence govern loading — project (`.pi/observers/`, trusted only), user (`~/.pi/agent/observers/`), and builtin — all keyed on the `name:` field, not the filename. Project-layer observers only load when the project is trusted, because an observer definition is an agent that runs on the user's credentials at a trigger the file chooses. When skipped, a warning surfaces in both session-start output and `/observers`.

### The Reconciler

The reconciler (`src/reconciler.ts`, 360 lines) is the central arbiter. It accepts or rejects proposals using:

- **Deduplication by fingerprint:** Advisories with the same fingerprint are delivered once per session. Within a single batch, same-fingerprint advisories are collapsed, keeping only the highest-priority one.
- **Priority ranking:** Higher-priority proposals sort first.
- **Per-turn advisory cap:** `maxAdvisoriesPerTurn` (default 2, max 10). Over-cap advisories are deferred, not dropped — released into the next turn's budget as a FIFO queue bounded at 100 entries.
- **Veto budget with three ceilings:** Per-fingerprint budget (`vetoBudget`, default 3), per-observer ceiling (`vetoBudget × 2`), and session-wide ceiling (`vetoBudget × 4`). The ceilings exist because the fingerprint is chosen by the observer's model — a model that varies its fingerprint gets a fresh budget every time. The ceilings, keyed on the observer name (from the definition file, not the model), actually terminate the loop.

The composite spend key (`vetoKey()`) uses length-prefixed encoding (`observer.length + "\0" + observer + fingerprint`) rather than delimiter-joining, preventing collision-based budget refunds where observer "a" + fingerprint "b:c" produces the same key as observer "a:b" + fingerprint "c".

### Arrival-Driven Delivery

Proposals are delivered the moment they're ready through pi's message queues — `steer` for advisories (appended before the next model call) and `followUp` for vetoes and settlement. This replaces an earlier fixed-drain-point design. Key implications:

- A veto formed while the agent is still working holds the run open — the agent addresses the unmet goal inside the same run.
- `skill-recall` can steer its suggestion into the run it read (at `before_agent_start`), rather than always landing one request late.
- Advice that arrives while the session is idle is appended immediately — closing the session no longer discards it.

What remains: a session that ends mid-run discards in-flight observer runs (the non-blocking design refuses to hold shutdown open on a model call), and a single-round-trip answer that finishes faster than any observer's model call (1.4–5.5s) cannot receive advice inside that run.

### Hermetic Sessions

Each observer gets a nested `AgentSession` with `noExtensions`, `noSkills`, `noPromptTemplates`, and an in-memory `SettingsManager` with `projectTrusted: false`. This prevents recursive observer spawning and isolates observers from the host project's settings, compaction policy, retry configuration, and thinking budgets — all of which could silently change observer behavior or mask failures.

The observer's system prompt is wrapped in fixed framing that establishes the read-only, propose-or-stay-silent contract. The observer's own prompt body is quoted inside unforgeable section markers. Tool objects are created once (not per-run); the output collector is reset per run.

### Non-Blocking Bus

`ProposalBus` (`src/bus.ts`, 131 lines) schedules observer runs without awaiting them. `kick()` returns immediately — each run races against its timeout via `Promise.race`, guaranteeing slot release regardless of run behavior. Observers accumulate three-strike disable on consecutive failures. Overlapping runs on the same observer throw (to prevent misattributed proposals).

## Key Techniques

### Sanitized Brand Type (`src/slices.ts`)

A TypeScript brand type that enforces prompt injection defenses at compile time. Only values that have passed through sanitization can enter assembled documents. Key mechanisms:

- **Unforgeable section markers:** A run of "=" characters one longer than the longest such run in any rendered body. No attacker-controlled content can contain a boundary line, regardless of what characters it uses.
- **Backtick fence as defense-in-depth:** If the unforgeable marker somehow fails, the markdown backtick fence contains the damage.
- **Head-and-tail truncation:** Tool call and skill collections are truncated at both ends, closing both positional attacks (content at start and end of a collection).
- **LINE_SEPARATOR_CHARS:** Ten Unicode codepoints that all major consumers treat as line breaks — `oneLine()` collapses them all, not just `\n`.
- **Compile-time assertion:** `UNBRAND_REQUIRES_SANITIZED` prevents the `unbrand()` helper from accepting a plain string — it only accepts `Sanitized`, so there is no accidental path from untrusted to trusted.

### Veto Budget Ceilings

The budget system is designed for an adversarial model where the observer's own fingerprint (chosen by its model, which reads a repo-resident prompt) cannot be trusted to terminate a veto loop. Three ceilings, all derived from a single `vetoBudget` setting (capped at 10):

1. **Per-fingerprint budget:** `vetoBudget` (default 3). Controls repeated vetoes of the same goal.
2. **Per-observer ceiling:** `vetoBudget × 2` (default 6). Keys on the observer name from the definition file — not model-chosen.
3. **Session-wide ceiling:** `vetoBudget × 4` (default 12). Keys on total veto count across all observers. Exists because nothing bounds the number of veto-capable observers a project can define.

The ceilings are derived, not separately configurable, so that the cap of 10 on `vetoBudget` hard-caps them all at once. A free-standing ceiling setting would lose that property.

### Fingerprint Re-validation on Restore

When rebuilding reconciler state from session entries after `/reload`, every replayed fingerprint is re-validated — over-long fingerprints (>512 chars) are rejected, not truncated. Truncation would map two distinct advisories sharing a long prefix onto one key, silently suppressing the second. Rejection means an advisory with an absurd fingerprint may be delivered again, which is visible and bounded by the per-turn advisory cap.

Crucially, the per-observer ceiling is credited from the replayed observer name alone (fingerprint validity gates only the fingerprint-keyed budget), because the ceiling exists to be the part the model cannot influence. A model choosing a 5,000-character fingerprint cannot refund its ceiling by making the fingerprint unreplayable.

### Model Resolution Chain

Observers resolve models through a 6-step chain, stopping at the first hit:

1. The observer's `model:` field, or `defaultModel` if absent
2. Each entry in `fallback:`, in order
3. The session's own model (whatever pi is running)
4. Disable (observer is silent, reason shown in `/observers`)

Steps 1–2 each try exact match, then fuzzy match (./- equivalence, date stamp stripping), then any-provider. A provider is only considered if it has configured auth — naming a model you have no credentials for falls through rather than resolving to something that can't run. The gap: auth ≠ working. A retired model ID with auth configured resolves successfully, fails every run, and is disabled after three strikes.

## Design Decisions

**Observers are files, not code.** A Markdown file with YAML frontmatter is an observer. No TypeScript required (though the extension itself is TypeScript). This puts observer authoring within reach of anyone who can write a system prompt, while the extension's implementation handles the hard parts (session isolation, injection defense, budget enforcement).

**Non-blocking by design.** `bus.kick()` returns immediately. Observer runs are fire-and-forget. This means a session that ends mid-run discards in-flight results, but it guarantees observers never slow down the agent's main loop. The trade-off is explicit: throughput over completeness.

**Precedence, not merging.** A project definition with `name: goal-tracker` completely replaces the bundled one — prompt, trigger, model, permissions. Overriding one observer never disturbs others. This is a simpler and more predictable model than field-level merging.

**One emission per observer per run.** The output tools (`propose`, `veto`) validate before consuming: blank fingerprints rejected, over-length text rejected, duplicate calls within a single run rejected. The observer gets one shot per wake — the tool is consumed, not reusable.

**Trust is gated at the project layer.** Project `.pi/observers/` only loads when the project is trusted. User-level observers (in `~/.pi/agent/observers/`) load unconditionally. This is the right boundary: you trust yourself, you don't trust a cloned repo by default.

## Comparison Notes

**vs. Claude Code hooks:** Claude Code's hook system (PreToolUse, PostToolUse, Notification) is event-driven and synchronous — hooks fire at specific tool-call boundaries and can block execution. Pi Observers are asynchronous and advisory: they propose, the reconciler decides, and the observer never blocks the main loop. Hooks are deterministic policy enforcement; observers are probabilistic quality monitoring. They're complementary: hooks catch known failure modes, observers catch novel ones.

**vs. The Advisor Strategy:** [[The Advisor Strategy]] inverts the orchestrator-worker pattern — a cheap executor escalates to an expensive advisor on hard decisions. Pi Observers inverts the same pattern differently: cheap background watchers (Haiku, default) propose to a reconciler that gates what the expensive main agent sees. Both are bottom-up escalation, but observers are persistent and trigger on lifecycle events, while the advisor is invoked at the executor's discretion. The observer pattern is "always watching"; the advisor pattern is "called when needed."

**vs. [[Guardrails and Feedback Loops]]:** Observers operate at the context level (advice injected into the session), not at the enforcement level (blocks that prevent execution). They're prompt-level guidance with structural guarantees (budgets, ceilings, dedup) rather than deterministic enforcement. An observer can suggest; it cannot block a tool call. The `goal-tracker` observer's veto is the exception — it holds the run open but doesn't block individual tool calls. This positions observers between soft prompt guidance and hard hook enforcement in the [[Harness Engineering]] spectrum.

**vs. Agent orchestration patterns:** [[Agent Orchestration]] catalogs coordination patterns — planner/worker/judge, hierarchical decomposition, role-based teams. Pi Observers is a different pattern: it's not coordination (agents working together) but monitoring (agents watching and commenting). Observers don't participate in the work; they're a quality-control layer orthogonal to the work. The closest analogue is the judge role, but a judge evaluates completed work; observers run continuously and may interrupt mid-run.

**vs. [[Components of a Coding Agent]]:** Raschka's six components describe the harness scaffold. Pi Observers is an extension that adds a seventh concern: background monitoring and advisory injection. It's a harness component, not a coding agent — it doesn't write code, it watches the agent that does. The Sanitized type system is a direct implementation of component 3 (structured tools, validation, permissions) applied to the observer's own rendering pipeline.

---

*Sources: [[raw/pi-observers]], [[summary/pi-observers]]*
*Last updated: 2026-08-07*
