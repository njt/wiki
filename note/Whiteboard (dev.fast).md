# Whiteboard (dev.fast)

Whiteboard (repo internally "Review") is an MIT-licensed desktop app where coding agents and humans co-author architecture documents on a shared canvas. The agent — Claude Code, Codex, Cursor, pi — is given an MCP tool surface that lets it write typed blocks (sequence diagrams, flow diagrams, ER "database lenses", call-stack diffs, code peeks, trace quotes) into a live document the human reads and clicks through. It inverts the usual review direction: instead of the agent reviewing code the human wrote, the agent *explains* the code it wrote, with every claim hyperlinked to real lines.

---

## Architecture

A pnpm monorepo with a clean protocol/package split:

- **`packages/review`** — the bulk (~84k lines of TS): the `review`/`whiteboard` CLI (`src/cli.ts`), the authoring engine (`src/authoring.ts`, 1,371 lines), the review HTTP API (`src/review-api/`, 64 modules including an MCP server at `review-api/mcp.ts`), document materialization, migration, telemetry, and sharing.
- **`packages/review-protocol`** — the wire contract: Zod schemas for the document, diff sides (`base`/`head`), views (`review | commits | diff | map | trace`), and schema versioning (currently `REVIEW_SCHEMA_VERSION = 5`). Versioned, explicitly-annotated schemas between CLI and desktop — the discipline most agent tools skip.
- **`packages/trace-core` + `trace-protocol`** — agent-session capture: parsers per harness (`claude-code`, `codex`, `opencode`, `pi`, `unknown`), a normalized event schema (`user | assistant | tool | separator` with per-tool additions/deletions), git-hook runners, and an S3/R2-or-hosted store with consent gating and auth split (writers upload; only admins read).
- **`packages/local-vcs`** — file-locked git operations, blob batching, working-diff reading against local checkouts.
- **`apps/review-desktop`** — a *vendored, patchless* Code-OSS fork. The README argues forks that maintain patches fail because "coding agents have a hard time with patches"; they merge upstream security/feature patches instead and note ~45% of stock Code-OSS is Copilot code they don't need.

Agent integration is per-harness plugins: `.claude-plugin`, `.codex-plugin`, `.cursor-plugin`, and a pi `SKILL.md` — thin shims over one MCP stdio server exposing authoring tools (`session_create`, `session_diff`, `session_set_target`, `session_activity_begin/update`, block writers) plus status/capability tools.

## Key techniques

- **Typed diagram blocks, not freeform drawing.** Each block type in `review-api/blocks/` is a Zod-validated shape with a `check()` invariant function — e.g. a sequence `step` must reference known actors and carry *exactly one* of source/explanation/code (`blocks/sequence.ts`); `call_stack_diff.ts` pairs `base`/`head` frame arrays by key with `via: {kind: call|queue|callback|rpc}` edges. The agent writes structured data; the app renders and verifies. Hallucinated diagrams are structurally impossible, only semantically wrong.
- **Served instructions as the prompting layer.** `session_get_instructions({topic})` returns the operating manual from the app itself (`packages/review/instructions/authoring.md`): the exact six-step flow, which diagram to choose for which change shape, "write incrementally — the user sees you write in real time", and the source-linking syntax `[label](review-source:head/src/file.ts#L10-L24)`. Prompt engineering ships as product content with version control, not as folklore in a README.
- **Source alignment from a Rust structural differ.** `@dev.fast/diffr` is a native/WASM AST diff; `review-protocol/src/source-alignment.ts` zips the two sides' diff leaves by `alignment_id` into a line-row table tuned for Monaco's trailing-newline quirks. This is what makes the semantic diff viewer — pseudocode summaries of big added functions, tests/docs collapsed — clickable line-for-line.
- **Trace archaeology.** Commits written by agents carry `Agent-Session: <id>` trailers. `whiteboard trace blame <file> -L start,end --json` resolves line provenance, `trace pull` fetches the session transcript into a local corpus laid out as `<owner>/<repo>/<session>/main.jsonl` (subagents get their own files), and the instructions teach FFF search with the "event index = line − 2" mapping. It's a provenance chain from a clicked line of code back to the exact tool call that produced it.
- **Decision-log self-instrumentation.** Agents call `session_activity_begin/update` to narrate their own progress on the canvas, and the trace views surface "what the agent decided autonomously" — the app is designed to make the agent's agency legible rather than hide it.

## Design decisions

- **Vendored Code-OSS, patchless.** They gave up fork hygiene for merge-ability, explicitly because agents write bad patches. Trade-off: heavy merge burden and Copilot-era bloat, in exchange for free LSP, keybindings, and diff rendering.
- **Rust for the diff, TS for everything else.** The one genuinely compute-bound piece (structural diffing across large files) is native; the block system, capture, and CLI stay in TypeScript. Sensible placement of the performance boundary.
- **Instructions served, not documented.** Rather than hoping agents read docs, the tool is only useful through its tool surface, which routes every agent through `session_get_instructions` first. This is the same move as skills-as-product, enforced by architecture.
- **Consent-gated trace capture.** Full transcripts go to a chosen origin only after explicit `trace allow`, with role separation (writers can't read others' sessions). They treat agent-transcript collection as the privacy-sensitive thing it is.
- **Honest limitations** listed in the README: no file editing in-app, single-repo reviews only, share snapshots don't live-update.

## Comparison notes

- Unlike [[6 Learnings from 12,000 Agentic Code Reviews]], which automates review as reviewer agents gating merges with no human reading code, Whiteboard automates the *explanation* side: the human is still the reader, and the artifact is understanding rather than a verdict.
- It complements [[Understand-Anything]], which builds persistent knowledge graphs of codebases; Whiteboard builds per-change, ephemeral, agent-authored documents pinned to commits, and grounds them in the same "click through to source" philosophy.
- Where [[FrontierCode]] measures whether agents' PRs would be *accepted*, Whiteboard targets the step after merge-worthiness: whether a human can *understand* what landed — the Geoffrey Litt "understanding is the bottleneck" thesis embodied as software.
- Against [[lat.md]] (knowledge graphs validated against drift), Whiteboard's decision log attacks drift from the other end: instead of validating a static graph, it binds each change to the agent-session traces that produced it.

#tool #project #agents #ai-code-review #developer-tools

---
*Sources: [[raw/whiteboard]], [[summary/whiteboard]]*
*Last updated: 2026-10-02*
