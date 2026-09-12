# Pi Subagents

Nico Bailon's open-source multi-agent extension for the Pi coding agent: a single `subagent` tool that delegates work to focused child Pi sessions with chain execution, sandboxed JavaScript workflow DSLs, async background jobs with a FleetView TUI, adversarial watchdog review, and an RPC protocol for external control. The most architecturally complete subagent extension for any open-source coding agent — it implements the same orchestration patterns as Claude Code's dynamic workflows but as a Pi extension rather than a platform feature.

---

## Architecture

The extension is a **single-tool delegation substrate** layered over Pi's extension API. Everything — chain execution, workflows, async jobs, RPC, watchdog — is reachable through one `subagent` tool with TypeBox-validated parameters. This is the opposite of the typical "many tools" approach: instead of separate tools for scout, review, chain, and workflow, a single tool with a rich parameter schema lets the parent model decide how to compose work. The model chooses agent type, task, execution mode, and safety constraints; the extension handles lifecycle.

### Key subsystems

**Agent system** (`src/agents/agents.ts`, ~1,890 lines): Loads agents from four sources with merge priority (project > user > package > builtin). Agents are markdown files with YAML frontmatter — the same filesystem-native pattern as Claude Code skills. Agent configs specify name, description, tools, model, thinking level, skills, and context strategy (fork vs. inherit).

**Chain execution** (`src/runs/foreground/chain-execution.ts`, ~1,527 lines): Three step types compose into arbitrary DAGs. Sequential steps run one agent after another. Parallel steps fan out concurrently with optional fail-fast. Dynamic parallel steps extract structured output from a prior step via JSON Pointer, then fan out N children — one per element — before collecting results. Checkpoint steps save intermediate state for resume. Every step supports acceptance gates, worktree isolation, and model overrides.

**Workflow scripts** (`src/workflows/scripted-workflow.ts`): Sandboxed JavaScript running in Node.js worker threads. The DSL provides `runs` (launch children), `state` (durable key-value store for missions), and gate commands (`!command`) for deterministic checkpoints between stages. This is a lighter-weight alternative to chain execution — less structured, more flexible, closer to Claude Code's dynamic workflow paradigm but without requiring the model to generate the JavaScript.

**RPC protocol** (`src/extension/rpc.ts`, ~654 lines): v1 protocol over events with seven methods. `spawn` launches children programmatically. `steer` sends mid-run corrections. `status` builds fleet snapshots aggregating foreground controls + async jobs into bounded entries (max 16 visible, 256 candidates). This is the integration surface for external tools like [[Herdr inspector integration]] and the FleetView TUI.

**Watchdog**: A separate model instance that inspects turn deltas between parent messages. Opt-in, adversarial — it's a reviewer that runs automatically rather than on-demand. Combined with LSP checks and child tool permissions, it creates a defense-in-depth safety layer that catches issues the primary model misses.

**Missions** (`docs/missions.md`): Durable goal tracking with lifecycle — missions persist across sessions with delivery receipts. Timed and recurring runs let missions function as scheduled work items. This is the closest any Pi extension gets to `/loop` and `/goal` in Claude Code.

### The agent-as-markdown pattern

Agents are defined as `.md` files with YAML frontmatter. This is **convergent evolution** with Claude Code's skills system and omp's agent definitions — the filesystem as configuration database, markdown as the universal format. The difference: pi-subagents uses markdown for agent *definitions* (what an agent is), while Claude Code skills use it for *procedures* (what an agent does). Both share the insight that YAML + markdown is the right format for LLM-maintained configuration.

## Key Techniques

### Single-tool delegation surface

The most important architectural decision: one tool, not many. The `subagent` tool's TypeBox schema defines every parameter the model might need — agent type, task, chain steps, parallel tasks, dynamic expansion, acceptance gates, budgets, model overrides, worktree isolation, workflow scripts. The model composes these parameters into the right call for the task. Compare to Claude Code's approach with separate primitives (subagents, dynamic workflows, skills as subagents) — pi-subagents unifies them into one tool call, trading API surface area for parameter complexity.

### Agent sourcing with merge priority

The four-tier agent source hierarchy (project > user > package > builtin) means teams can override built-in agents per-project while individual developers can override per-machine. External agent dirs via `PI_SUBAGENT_EXTRA_AGENT_DIRS` env var (PATH-separated) add a fifth dimension. This is the same pattern Claude Code uses for skills (enterprise > personal > project) but applied to agent definitions rather than procedures.

### Dynamic fan-out with JSON Pointer

Chain execution's dynamic parallel step extracts a JSON array from a prior step's structured output, fans out one child per element, and collects results. The JSON Pointer syntax (`/path/to/array`) references specific fields in prior outputs. This is the same fan-out pattern from Claude Code dynamic workflows but hardcoded into chain syntax rather than generated in JavaScript — less flexible but more predictable.

### Worktree isolation as a primitive

Both chain steps and workflow script runs support `worktree: true`, creating isolated git worktrees so parallel agents can mutate files without conflict. This is the same primitive Claude Code uses for dynamic workflow subagents and [[Fleet Supervisor (sermakarevich)]] uses for parallel task runners. The convergence across three independent implementations suggests worktree isolation is the right primitive for parallel agent execution.

### Capability ceilings

Child agents can be restricted below parent capabilities: limited tools, models, turn budgets, tool budgets, usage budgets. This is a structured form of the quarantine pattern described in [[Dynamic Workflows in Claude Code]] — agents doing untrusted work get lower ceilings. The difference: pi-subagents makes ceilings explicit parameters rather than architectural patterns, which is both more composable and easier to forget.

### Intercom bridge

A dedicated channel for parent-child communication during execution. Children can escalate decisions, request clarification, or signal completion without ending the run. The parent can steer, interrupt, or provide additional context mid-flight. This is the same need that [[The Advisor Strategy]] addresses but as a bidirectional channel rather than a one-way escalation — the child can ask questions, not just receive orders.

## Design Decisions

**One tool, many modes.** The `subagent` tool is the sole entry point for all delegation. This is a bet that model composition of parameters beats human-designed APIs. It works well for the six built-in agents and straightforward chains; the complexity cost shows up in workflow scripts, where the model must generate JavaScript that runs correctly in a sandboxed worker thread. The chain syntax is complex enough that the README recommends natural-language requests ("Use reviewer to review this diff") rather than teaching users the parameter schema — the model translates intent to tool calls.

**Extension, not platform.** Unlike Claude Code's dynamic workflows, which are a platform feature, pi-subagents is a Pi extension. This means it has access to Pi's full extension API (lifecycle hooks, message renderers, event system) but not to the agent loop internals. The watchdog, for example, can inspect turn deltas because it hooks into lifecycle events — but it can't modify the agent's system prompt or change how tools execute. This is the right boundary: extensions extend behavior without modifying core.

**Foreground as default, async as opt-in.** The extension defaults to foreground execution (streaming progress inline) but supports async via simple natural language ("Run this in the background"). This is the opposite of [[Fleet Supervisor (sermakarevich)]], which is async-by-default with a queue. For interactive use, foreground is right; for production pipelines, async is right. pi-subagents serves both without forcing either.

**Markdown agents with YAML frontmatter.** The filesystem-native agent definition format — `.md` files with frontmatter — is the same pattern as Claude Code skills, omp agents, and [[The Agentic Product Standard v2.0]]'s skill definitions. It's becoming the universal format for LLM-maintained configuration: human-readable, machine-parseable, version-controllable. pi-subagents was an early adopter of this pattern in the Pi ecosystem.

**Opt-in safety, not default safety.** The watchdog is opt-in. Capability ceilings are per-call. Acceptance gates are per-step. This is a deliberate choice: make safety mechanisms available but not mandatory, trusting the user (or the parent model) to apply them appropriately. Compare to Claude Code's permission system, which is default-deny, or [[yolo-cage]], which enforces isolation at the container level. pi-subagents is more flexible but requires more judgment to use safely.

## Comparison Notes

**vs. Claude Code Dynamic Workflows**: pi-subagents implements the same orchestration patterns (fan-out, chains, adversarial review) as Claude Code's dynamic workflows, but as a Pi extension rather than a platform feature. The key architectural difference: Claude Code's workflows are LLM-authored JavaScript harnesses; pi-subagents' chains are declarative JSON with an optional JavaScript escape hatch for workflow scripts. Claude Code is more flexible; pi-subagents is more predictable. Both use worktree isolation as the primitive for parallel mutation.

**vs. [[Oh My Pi (omp)]]**: omp and pi-subagents are complementary Pi ecosystem components. omp is a fork of Pi-mono that goes maximalist on tools, providers, and native performance. pi-subagents is a Pi extension that adds multi-agent orchestration to the standard Pi runtime. They could theoretically compose: install pi-subagents on omp to get both omp's 32-tool surface and pi-subagents' delegation system. The practical question is whether omp's extension API is compatible — the README mentions pi-subagents as a distinct entity from the Pi fork lineage.

**vs. [[Fleet Supervisor (sermakarevich)]]**: Both are multi-agent supervisors, but at different levels. Fleet Supervisor is a standalone Python process that claims tasks from a queue and spawns coder subprocesses — it's infrastructure. pi-subagents is a Pi extension that adds delegation to an interactive session — it's a tool. Fleet Supervisor is async-by-default; pi-subagents is foreground-by-default. Fleet Supervisor works across coding agents (Claude Code, Codex, agy, OpenCode); pi-subagents works only within Pi. They address different scales: Fleet for production pipelines, pi-subagents for interactive development.

**vs. [[The Advisor Strategy]]**: pi-subagents' oracle agent is a manual advisor — you invoke it explicitly when you want a second opinion. Anthropic's advisor is automatic — the executor model decides when to consult. Both share the insight that specialist models in focused contexts outperform generalist models in broad contexts, but they automate differently. The intercom bridge in pi-subagents is more general than the advisor's one-way consultation — it supports bidirectional communication during execution.

**vs. [[Thrifty (Tiered Delegation for Claude Code)]]**: Thrifty and pi-subagents share the tiered-model economics insight but execute it differently. Thrifty hardcodes the tiering (Sonnet plans, Haiku builds, Sonnet checks failures); pi-subagents lets the parent model choose per-child models at invocation time. Thrifty's gates are deterministic re-runs (tests pass/fail); pi-subagents' acceptance gates have three levels of scrutiny. Thrifty is optimized for cost; pi-subagents is optimized for flexibility.

**vs. [[Claude Code Skills System]]**: Both use the filesystem-native markdown + YAML frontmatter format for configuration. Claude Code skills define procedures (what should happen); pi-subagents agents define capabilities (who should do it). The two formats are structurally identical — YAML frontmatter + markdown body — but semantically different. A Claude Code skill that defines a review procedure could be paired with a pi-subagents reviewer agent that executes it.

---

*Sources: [[raw/pi-subagents]]*
*Last updated: 2026-08-07*
