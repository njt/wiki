# Reticle — The Proof Layer for AI Agents

Reticle answers the question every agent-coding workflow eventually hits: the agent says "done," but who checks? It is a verification layer for web apps driven by coding agents — an in-page SDK that instruments the real running app, an MCP server the agent talks to, and a deterministic rules engine that returns `pass · fail · cannot tell` with the `file:line` to fix. Unlike browser-screenshot approaches, it reads the accessibility tree, network log, console, and framework state as structured data, and it grades every assertion by how independent the evidence is from the code being verified.

---

## Architecture

A pnpm + Turborepo monorepo (~4,100 files, v3.9.0) with hard package boundaries, each enforced by lint rules (`scripts/check-boundaries.mjs`):

- **`@reticlehq/core`** — the wire contract: every constant and zod schema crossing any boundary lives here once, so browser and server cannot drift. Every inbound WebSocket message is zod-parsed; malformed frames are dropped and recorded as gaps.
- **`@reticlehq/browser`** — the SDK embedded in the app in dev: `MutationObserver` for DOM, wrappers around `fetch`/`XHR`, console and history hooks, plus opt-in registries (`registerStore`, `reticle.signal`). Never imports Node APIs.
- **`@reticlehq/server`** — the WebSocket bridge, MCP server, CLI, and flow/run storage. Never imports DOM APIs. Exposes `reticle_look` / `reticle_act` / `reticle_assert` / `reticle_act_and_wait` as MCP tools.
- **`@reticlehq/engine`** (~16.6k lines) — the verdict rules, deliberately free of browser and daemon code. `src/evidence/` grades evidence; `src/disagreement/` (~50 test files) detects contradictions in a window: double submits, stale responses, echo mismatches, unit mismatches.
- **`open-verification/`** — the Open Verification Protocol (OVP): an RFC-2119-style spec with permanent requirement identifiers (`OVP-CHAN-2`), JSON schemas generated from source (never hand-written, to kill drift), and conformance vectors.
- **`@reticlehq/init`** — zero-install project detection: lockfile sniffing, dev-server port convergence, framework adapters for Next, Vite, Angular, Nuxt, Django, Streamlit, Electron, Tauri.
- **`skills/`** — 21 agent skills (`verify-login-logout`, `false-green-tests`, `verify-optimistic-update`, …) that package verification recipes for agents.

Data flow: agent → MCP tool call → server → WebSocket command → SDK executes in the live page → structured events stream back through a bounded ring buffer → predicates evaluate over program truth → verdict artifact written under `.reticle/` (flows, baselines, runs, and a versioned capabilities contract).

## Key techniques

- **Tiered evidence.** Tier 1: an app-emitted signal (`reticle.signal('order:placed')`) — a wrong element can't fake it. Tier 2: network + route + framework state. Tier 3: DOM/text presence, the weak fallback the tool nudges away from. The verdict always states which tier it used, so "green" carries its own confidence.
- **Provenance and independence.** OVP's load-bearing rule: an observation must record whether its source could have been influenced by the action it is evidence for. The app's self-report of success is graded accordingly.
- **Record once, replay with no model.** Flows are recorded by the agent once; CI replays them deterministically by re-resolving durable element anchors against the live DOM — a couple hundred tokens instead of a full agent drive. Self-healing rebinds a drifted anchor only if the declared consequence still fires, so it never "heals to the wrong element and ships the regression."
- **Contradiction detection.** The engine doesn't just check predicates; it folds a window of events looking for internal disagreements — a poll that isn't a double submit, a stale response served after a newer one, a UI acknowledging a write whose network call failed.
- **Subject identity.** OVP distinguishes `instance` (dies on navigation — a total invalidation) from `epoch` (advances on hot edits — a partial one), so evidence is invalidated with the correct granularity rather than silently outliving the thing it described.
- **Boundary-linting itself.** The repo tests its own discipline: `orphan-modules.test.ts`, `directory-reach.test.ts`, `import-graph.test.ts`, `the-rules-stand-alone.test.ts` — tests that verify the architecture rules hold.

## Design decisions

- **Structured reads over screenshots.** A screenshot costs ~1,365 image tokens per look and is blind to a failed network request. Reticle trades pixel fidelity for cheap, deterministic, non-visual evidence — the same bet as [[Computer Use is 45x More Expensive Than Structured APIs]].
- **Dev-only by default, security-conscious.** The SDK is tree-shaken out of production builds; the bridge binds to loopback, requires a pairing token, and refuses non-loopback hosts unless origins are explicitly allowed. Every security-gating env var is a single named constant so a typo can't silently disable auth.
- **Three-valued logic.** `yes · no · cannot tell` — "cannot tell" is a first-class verdict backed by blind-spot detection (`evidence/blind-spots.ts`, `gap-ledger.ts`), an honest answer most tools collapse into "pass."
- **Trade-off**: the tier-1 signal requires a ~30-second app opt-in, so the strongest evidence needs the app author's cooperation; tier 3 is always available but weak. Reticle is honest about landing on the weak tier rather than pretending otherwise.

## Comparison notes

- Unlike [[Chrome DevTools MCP — Debug Your Browser Session]], which gives an agent raw debugging access to a browser, Reticle is opinionated about what counts as proof: it grades assertions and refuses to let DOM presence stand in for a consequence.
- It complements [[Sixteen Agents and a Byte-Exact Oracle]]: that project's lesson was that a cheap, exact oracle beats an expensive model-judge; Reticle industrialises the same insight for web apps, with recorded flows as the oracle and replay as the byte-exact check.
- It is the systems-level counterpart to [[The Oracle Is the Asset]] — Sam Ruby's claim that the test suite is the durable asset. Reticle makes the oracle a versioned, git-checked artifact (`.reticle/`) with a frozen capabilities contract.
- Compared with [[Proving It Works]] (narrated demo movies checked mechanically), Reticle shares the "mechanical check on the agent's claim" goal but chooses structured program truth over sampled picture-and-sound, at much lower token cost.

#tool #project #agents #testing #verification
---
*Sources: [[raw/reticle]], [[summary/reticle]]*
*Last updated: 2026-11-13*
