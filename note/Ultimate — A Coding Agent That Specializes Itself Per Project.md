# Ultimate — A Coding Agent That Specializes Itself Per Project

Ultimate (Daniel Griesser, MIT, github.com/HazAT/ultimate) is a local coding agent with a web UI built on the Pi stack (@earendil-works/pi-coding-agent, pi-durable, pi-codemode, pi-claude-code). Its thesis: the agent core should be identical everywhere, and the project-specific agent should be built *by the agent* into a committed `.ultimate/` folder — custom tools, panels, subagents, skills, even whole full-screen app "surfaces" — hot-reloading while you talk. The unusual bet is that this is only safe if the whole system is durable and self-hosting: everything commits to SQLite before display, so Ultimate can rewrite its own UI, extensions, and core mid-conversation without losing its place.

---

## Architecture

Process topology from `bin/ultimate.ts` (the supervisor) and `server/main.ts` (~330 lines):

- **Supervisor** (`bin/`) is deliberately tiny — AGENTS.md says changes there require a manual human restart. It forks a core child process, holds a lock file per project state home, opens the browser as a chromeless `ultimate.localhost` app window, and renders a terminal dashboard of lifecycle state.
- **Core server** (`server/main.ts`) wires an HTTP server, a middleware-mode Vite dev server (the web UI is hot-reloaded dev-server React, not a build), extension loading (`server/extensions.ts`, 833 lines), and Pi Durable storage. Supervised mode speaks a small `supervisor-protocol.ts` message protocol (`go`/`release`/`exit`), so the supervisor can stop and start the serving layer without killing storage.
- **Everything else is an extension**: ~55 folders under `extensions/`, each with `server.ts` (agent behaviour: system prompt, tools, hooks) and/or `web.tsx` (UI: slots, panels, surfaces). The largest are `lineage` (1,656 lines — the conversation-tree/memory machinery), `subagents` (1,228), `automations` (968), and `codemode` (826). One-way dependency tiers are enforced by lint (`lint/ultimate.ts`): a `tier = 'core'` extension may only depend on other core-tier extensions, and `pnpm deps` fails on cycles.
- **Three reload speeds** (AGENTS.md's layer table): `web.tsx` hot-reloads instantly via Vite; `server.ts` reinstalls on save, with running tool calls finishing on old code; `server/`+`shared/` core changes go through a `restart` tool that boots and type-checks a new process beside the old one — failed boot rolls back, success resumes the run in the new generation. `bin/` changes need a human.

Specialization is layered: the clone's built-ins, then `~/.ultimate/` (personal), then `<project>/.ultimate/` (shared with the repo; later names win). Project extensions run code, so Ultimate asks once before adopting a project's setup. State is per-project in `~/.ultimate/projects/<name>-<hash>/state.sqlite`.

## Key techniques

- **Durable-first persistence** (docs/durability.md): every conversation, tool call, subagent, and document is committed to SQLite *before it is shown*; a restart or crash resumes from the last checkpoint. Tools declared `replay: "safe"` rerun after a crash (with a mandatory reason comment, enforced by lint); all other interrupted tools report "interrupted" to the model. Durable events commit in the same transaction as the change they describe, and a reactor's cursor commits with its handler's writes — exactly-once in-transaction, at-least-once outside.
- **The forest of conversation trees** (`extensions/lineage/`): one root per project runs no turns; it holds a "tally" — one intent line per branch, each conversation's latest abstract, landings with divergence — plus slug-indexed project memories. Branches fork the root (optionally into isolated worktrees) and start from a *memory brief* rather than the full tally; abstracts are "liquid": the first summary becomes an anchor entry and later summaries are carriers that edit the anchor's text in place, so the tally never reorders while staying append-only. Subagents are explicitly *not* branches — branches plan/decide/verify, subagents do bounded execution, and a "lander" is the final gate to main.
- **Multi-instance as a feature**: every project is its own process and state home. Instances discover each other via manifest files and live supervisor URLs (`ultimate.projects()` in codemode), read each other's conversations read-only (`ultimate.entries/search/trace`), message each other (`ultimate.tell`), report health via a `status` action, and reach other machines through a Tailscale extension — enabling a hub project whose @ultimate checks and starts work in the others.
- **Codemode** (`extensions/codemode/`): a sandboxed script runtime over the agent's own state — the README suggests "use codemode to check which of your extensions failed to load," i.e., the agent introspects its own runtime with code rather than bespoke tools.
- **Claude Code as a provider** (`extensions/claude-code/`): the agent's model backend can be Claude Code's CLI protocol and account, with `claude auth login` as the credential path — a reminder that harness and model are cleanly separable.

## Design decisions

- **Malleability over stability**: the README openly promises breaking changes "without migration paths or regard for anyone building on top," and PRs mostly stay open as reference ideas. This is a personal artifact used as a design document — the opposite of a platform's backward-compatibility contract, and honest about it.
- **Behaviour in prompts, not code**: AGENTS.md states rules about how agents behave live in prompts and agent definitions; code checks protect only safety and invalid states. Custom lint rules (`no-silent-catch` with required reason comments, `no-test-timeout`, `no-retired-ui`) enforce hygiene without policing judgement.
- **Reloadability as the organizing constraint**: the file layout is chosen almost entirely by "how fast can a change go live," which is why AGENTS.md says put behaviour in extensions rather than server/. It's a hot-reload-first architecture where the conversation must never die.
- **Trust boundary at the project**: adopting `.ultimate/` is an explicit one-time consent, since project extensions execute arbitrary code — a lightweight but real answer to the supply-chain problem of shared agent configs.

## Comparison notes

- It sits in the same family as [[TEHAS — A Pi Software Factory]] and [[pi-multiloop]]: Pi-based, extension-driven, and treating the Pi core as a substrate rather than a product boundary. Ultimate is the most ambitious of the three — it makes per-project self-specialization and self-hosting (the agent editing its own running server) the headline, whereas multiloop stays a single extension concern.
- Its tally-and-memory-brief design complicates [[Devin Memory and Dreaming]]: Devin consolidates lessons asynchronously in a Git-backed Memory Drive, while Ultimate keeps a continuously maintained *read-only context* (the tally) that branches inherit plus liquid, anchor-editing abstracts — memory is the tree's shared canopy, not a separate store. Both reject "just retrieve embeddings" in favour of curated, model-written summaries.
- Against [[The Log Is the Agent]]-style thinking, Ultimate is a strong endorsement: the SQLite log committed-before-display is what makes self-rewrite, crash resume, and cross-instance reads possible — the log literally is the running agent.
- Compared with [[Paseo]], which unifies many agent backends in one client, Ultimate inverts the problem: one backend (Pi), many per-project shapes. And unlike [[pi-multiloop]]'s compound verifiers, quality here is delegated to a standing lander gate plus review agents — review-as-institution rather than review-as-mechanism.

---

*Sources: [[raw/ultimate]], [[summary/ultimate]]*
*Last updated: 2026-11-02*
