# Open Design

A local-first design workspace where 16 different coding agent CLIs (Claude Code, Codex, Gemini, etc.) generate design artifacts — websites, prototypes, slide decks, images, videos — through a skill pipeline tempered by design systems, with a multi-panel critique jury that auto-converges on quality thresholds. The most ambitious attempt to treat coding agents as a commodity runtime for design generation.

---

## Architecture

Open Design is a three-layer local-first application split across `apps/daemon`, `apps/web`, and `apps/desktop`.

**Daemon** (`apps/daemon/`) is the sole privileged local runtime — an Express server that owns SQLite state, filesystem access, agent CLI spawning, and SSE streaming. It runs on localhost and is the only component allowed to touch `.od` state, spawn processes, or access the filesystem.

**Web** (`apps/web/`) is a Next.js 16 React frontend with a thin BFF/proxy layer. It communicates with the daemon exclusively through REST + SSE, never directly accessing files or processes.

**Shared boundary** lives in `packages/contracts/` — pure TypeScript DTOs, SSE event unions, error codes, and API types. This package has zero framework dependencies (no Next.js, Express, Node APIs). The architecture boundary is explicitly documented in `specs/current/architecture-boundaries.md` and enforced via `AGENTS.md` files at each directory level.

**Agent abstraction** (`apps/daemon/src/runtimes/`) supports 16 CLI agents via per-adapter definitions. Each `RuntimeAgentDef` specifies binary detection, auth probing, model listing, argument construction, and stream parsing. Detection handles nvm/fnm/mise PATH shim chains by probing at the resolved launch path, not the shim.

**Plugin pipeline** (`apps/daemon/src/plugins/pipeline.ts`) implements a headless scheduler with `until`-condition devloops — stages iterate with a parsed condition language, capped at `OD_MAX_DEVLOOP_ITERATIONS` (default 10), recording every iteration to SQLite.

**Critique Theater** (`apps/daemon/src/critique/`) runs a five-panelist Design Jury (Designer, Critic, Brand, A11y, Copy) in a single CLI session, with weighted composite scoring and auto-converging rounds. Default ship threshold: 8.0/10.

**Desktop** (`apps/desktop/`, `apps/packaged/`) uses a sidecar IPC pattern with POSIX sockets at `/tmp/open-design/ipc/<namespace>/<app>.sock`. The sidecar protocol (`packages/sidecar-proto/`) enforces five-field process stamps: `app`, `mode`, `namespace`, `ipc`, `source`.

## Key Techniques

**Agent-agnostic runtime layer** (`apps/daemon/src/runtimes/defs/`): Each CLI adapter defines fallback bins, CLI-probed capability detection (parsing `--help` output for optional flags like `--add-dir` and `--include-partial-messages`), stdin prompt delivery to avoid Windows `ENAMETOOLONG` (32KB CreateProcess limit), and `stream-json` input format for tool_result injection without re-spawning. Platform-specific quirks are localized: Codex uses `--sandbox danger-full-access` on Windows vs `workspace-write` on macOS/Linux.

**Until-condition devloops** (`apps/daemon/src/plugins/pipeline.ts`, `apps/daemon/src/plugins/until.ts`): Pipeline stages declare `repeat: true` + an `until` expression. The scheduler evaluates returned signals against the expression, capping iterations. This gives plugins looping behavior without unbounded API spend.

**Single-session multi-panelist critique** (`specs/current/critique-theater.md`): Five roles in one CLI invocation means the Designer's draft, Critic's notes, and Copy's edits all share one model context window — no cross-process trace correlation needed. The trade-off is no parallel panelist execution, estimated to save 2-3 weeks of implementation.

**Plugin registry protocol** (`packages/registry-protocol/src/schemas.ts`): Full Zod-schematized registry with versioning, distribution types (GitHub releases, HTTPS archives, local, database), signature verification (GitHub OIDC, Cosign, Minisign), yanking, and doctor checks. An npm-like package system for design tools.

**BoundedJson validation** (`packages/contracts/src/common.ts`): Live artifacts enforce explicit structural bounds (max 8 depth, 100 keys/object, 500 items/array, 16KB strings, 256KB serialized) rather than ad-hoc size caps.

**Filesystem-backed memory** (`apps/daemon/src/memory.ts`): Modeled after Claude Code's auto-memory pattern — `MEMORY.md` index + per-fact markdown files with YAML frontmatter. SSE EventEmitter pushes changes to the web UI in real time.

## Design Decisions

**Simplicity over parallelism.** Single CLI session per artifact, even for multi-round critique. The critique spec explicitly rejects parallel panelist processes, arguing coherent model context outweighs parallel speed.

**Express, not Fastify.** Chosen for development momentum. The maintainability roadmap defers Fastify evaluation until TypeScript enforcement, contracts, validation, tests, and modularization are in place.

**Filesystem, not database.** Artifacts, memory, and live artifacts live under `.od/` as files, not in SQLite. Users get direct filesystem access; query flexibility is sacrificed.

**Runtime detection, not static config.** Agent capabilities are probed at daemon startup via actual CLI invocation. Makes the UI self-healing on reinstall but adds startup latency.

**TypeScript contracts, not runtime schemas.** The web/daemon boundary uses TypeScript types. The roadmap (W4) acknowledges this gap: "type correctness alone cannot protect against malformed runtime input."

**10,028-line server.ts.** Acknowledged as P1 maintainability risk. Being split into `routes/`, `services/`, `agents/`, `db/`, `fs/`, `streams/`, `artifacts/` per the roadmap (W5).

## Comparison Notes

Unlike [[Gas Town's Agent Patterns]] which embraces chaotic multi-agent orchestration, Open Design takes the opposite approach: structured hierarchical roles and a single session. Its critique jury maps directly to Cursor's finding in [[Scaling Long-Running Agents]] that planner/worker/judge beats flat coordination.

The plugin pipeline with feedforward `until` conditions and critique jury as inferential feedback implements the framework described in [[Harness Engineering]], applied to design output rather than code.

Like [[Mission Control — Bhanu's 10-Agent Squad on OpenClaw]], it uses file-based memory and multi-agent patterns, but runs agents in one session instead of ten specialized concurrent agents.

The 154 design systems act as a spec layer, similar to the premise of [[Spec-Driven Development]] — a DESIGN.md constraining visual generation the way a spec.md constrains code.

The agent-agnostic abstraction (16 CLIs, one interface) is most comparable to [[acpx]], which wraps 16+ coding agents behind a single command surface.

#tool #project #agents #design-system #local-first #plugin-architecture

---
*Sources: [[summary/open-design]]*
*Last updated: 2026-05-18*
