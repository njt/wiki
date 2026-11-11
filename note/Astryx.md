# Astryx

Meta's open source design system, extracted after eight years of internal use (13,000+ apps, 150+ components) and rebuilt around a thesis: a design system's real customers are now humans *and* coding agents, so the API, docs, and CLI must be one coherent machine-readable surface. The code shows that thesis taken seriously as architecture, not marketing — the CLI is the primary interface, agent instruction files are generated and hash-managed, and even the naming conventions are empirically tested against LLMs.

---

Astryx is a pnpm monorepo (`packages/core`, `packages/cli`, `packages/build`, `packages/themes`, plus `apps/` for the docsite/Storybook and `internal/` tooling) on React 19 + StyleX. Components are plain TypeScript with colocated `.doc.mjs` documentation modules; themes are CSS custom property sets; consumers get pre-built CSS and typed components with no build plugin required — StyleX is authored internally but invisible downstream, and overrides happen through `className` (Tailwind, CSS modules, plain CSS all work).

## Architecture

- **CLI-first discovery** (`packages/cli/clients/cli/`): the CLI is ~43k lines across commands — `search`, `component`, `docs`, `template`, `build-theme`, `swizzle`, `init`, `discover`, `doctor`, `upgrade`, `hook`. It is explicitly "the primary interface for working with the design system, for humans and machines alike" (`packages/cli/README.md`). Every command has three layers: a colocated `*.doc.mjs` `CommandDoc` defining the command surface, a thin command handler that parses flags and renders, and a programmatic API function under `packages/cli/api/` returning the same typed `{ type, data }` envelope that `--json` prints. Agents and build tools import the same function the CLI wraps.
- **Global/project release handoff**: the bin (`bin/astryx.mjs`) resolves the *project's* version of Astryx even when invoked from a global install — a release/handoff module (`foundation/release/handoff.mjs`, spec AST-081) picks which release answers the current folder before any project config loads. This solves the "which version does the agent see" problem without pinning.
- **Agent-doc state as a canonical contract** (`foundation/agent-docs/agent-doc-state.mjs`): a deliberately dependency-free leaf module that knows where Astryx writes its agent block (AGENTS.md, CLAUDE.md, `.cursorrules`, GEMINI.md, HERMES.md…) and whether `init` has run. It is the single source of truth for three enforcement layers (postinstall nudge, per-command nudge, init command), and Core ships a *byte-identical generated copy* verified by `check:setup-contract`.
- **Knowledge-record governance** (`docs/specs/AST-*/spec.md`, ~40+ specs): records carry `authority: draft|current|archived`, owners, and `affects_*` routing. Only `current` records govern implementation; AGENTS.md instructs agents to run `pnpm specs:for <path>` before editing any path and route review claims through semantic triggers with preserved "review receipts."
- **Vibe tests** (`internal/vibe-tests/`): structured evals comparing LLM code generation across design-system configurations — a prompt battery (`test-sets/default.json`) with categories and expected components (used for scoring only), multiple "arms" testing alternative API namings (e.g. `edge/inset/flush/outset/bleed` vs `padded/flush/outset/bleed` for List), and probes for *use*, *recall*, and *reading* (does the name alone predict the rendering?).

## Key techniques

- **Hash-managed skill injection**: the Claude/Codex plugin (`plugins/astryx/skills/astryx/SKILL.md`) embeds an `astryx-managed sha256:...` marker so the CLI can detect and update its own block inside agent instruction files without clobbering human-written content — an idempotent, verifiable doc-update protocol.
- **Specs as review authority**: the review workflow derives semantic triggers per component, loads global baseline claims by commit range (`scripts/review-global-baselines.mjs --authority-commit <sha>`), and requires rubric results (`docs/contributing/spec-writing.md` R1–R7) in the PR description. A spec-writing rubric explicitly bans narrative ("we decided", PR numbers, project status) so records stay true "six months later."
- **StyleX capability scanning** (`internal/stylex-capabilities/scan.mjs`): an auto-generated matrix of which CSS features StyleX actually supports (88/92 at version 0.19.1), turning a styling library's limits into checked, regenerable documentation.
- **Command doc separation**: `defineCommand` splits command surface (doc), action (handler), and API — the same `search.doc.mjs` drives CLI help, typed JSON schemas, and API docs, so agent-facing and human-facing interfaces cannot drift.
- **Variants as open maps**: `ButtonVariantMap` is an interface themes extend via TypeScript module augmentation — customization happens in the type system rather than by wrapping components.

## Design decisions

- **Guidance over enforcement**: components render what you pass; design opinions live in docs. This inverts the usual design-system instinct (runtime guardrails), on the theory that both humans and agents churn against APIs that fight them.
- **No styling lock-in**: StyleX internally, anything externally. They paid the complexity of shipping pre-built CSS to avoid forcing a styling adoption decision — the single most common rejection reason for design systems.
- **Swizzle as an escape hatch**: ejecting component source into the consumer's project trades long-term upgrade safety for total control, made explicit rather than hacked around.
- **Docs read through the CLI, not the filesystem**: the skill instructs agents to run `astryx component <Name>` rather than open package files — the CLI is a controlled, version-correct context window, not just a convenience.
- **Vibe testing naming decisions**: API naming disputes are settled by running models against candidate arms and scoring correctness, invented props, and confidence calibration — an unusual institutionalization of empirical agent evaluation inside a UI library.

## Comparison notes

This is [[Design Systems for AI Agents (Luke Wroblewski)]]'s argument shipped as an entire product: Wroblewski proposes AGENTS.md plus tokens as the steering layer; Astryx goes much further, making the CLI the steering surface and generating the agent blocks from a single hashed contract so they can't drift. Where Wroblewski's docs stay current "by construction" because they're generated from code, Astryx adds the enforcement loop — `pnpm specs:for`, knowledge-record authority, `check:setup-contract` — that keeps the *steering layer itself* honest.

The `internal/vibe-tests` harness makes it a concrete data point for [[Guardrails and Feedback Loops]]: it's evaluation of agent output used not to gate a deployment but to *design an API*, closing a loop between model behavior and interface design that most eval write-ups in the wiki never consider.

And its spec/knowledge-record system — versioned, authority-stamped, rubric-checked records that AGENTS.md forces agents to consult before every edit — is the most industrialized instance in the wiki of [[Specifications as the Product]]: specs here are not planning artifacts but the durable, machine-routed contract that governs review.

For [[Software Engineering Craft]], the "guidance over enforcement" principle and the rubric that bans narrative from records are old ideas (conventions, present-tense documentation) made load-bearing by an audience that actually reads them every time.

---
*Sources: [[raw/astryx]], [[summary/astryx]]*
*Tags: #tool #project #design-systems #agents #frontend*
*Last updated: 2026-11-11*
