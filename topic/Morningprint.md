# Morningprint

A creative AI pipeline that turns a thermal receipt printer into a daily generative art installation. Each morning, Claude Fable 5 designs an original piece of CP437 character art themed to the day, a TypeScript renderer converts it to raw ESC/POS printer bytes, and a Raspberry Pi bridge prints one physical copy — 80mm wide, with a short verse underneath. No feeds, no refresh button, no screen: just paper and a corkboard. The project is a production case study in structured output as creative constraint, deterministic guardrails around untrusted model output, and dual-environment harness design where the local test tool mirrors production exactly.

---

## Architecture

The project is a two-node distributed system — cloud + kitchen — with a local development harness that replicates the production pipeline exactly:

**Cloud: Google Apps Script** (`src/art.ts`, 730 lines). A time-driven trigger calls `printDailyArt()`, which builds a context brief (`buildArtContext`), calls Claude Fable 5 via raw `UrlFetchApp` against the Anthropic Messages API (`generateDailyArt`), parses the structured-response JSON (`parseArtResponse`), validates it against length bounds (`validateArtSpec`), renders it to ESC/POS bytes (`renderDailyArtReceipt` → `renderArtSpec`), and POSTs the payload to the Pi bridge (`sendToPi` in `src/transport.ts`). All config lives in Apps Script Properties; no secrets in the repo.

**Kitchen: Raspberry Pi Zero W** (`docs/pi-print-server-runbook.md`). A ~40-line Python `http.server` receives the octet-stream POST, writes it directly to `/dev/usb/lp0`. Behind an ngrok static domain with basic auth, managed by systemd. The printer is an Epson TM-T20III, 80mm, 203 dpi, auto-cutter.

**Local harness** (`test-print.mjs`, 310 lines). Loads the exact production bundle (`dist/main.gs`) via `vm.createContext`, so `art` mode renders through the identical renderer as production. `art:live` mirrors the full Anthropic client including server-side web search and `pause_turn` tool loops. `--dry` prints the hex payload instead of sending. The `ruler` mode prints a calibration page that empirically confirmed the gapless line-spacing values (`ROW_DOTS_A = 24`, `ROW_DOTS_B = 17`).

**Build pipeline** (`build.js`, 43 lines). esbuild bundles the TypeScript import graph into a single IIFE (`dist/main.gs`) with a footer that re-exposes `printDailyArt` and `testDailyArt` as bare globals — Apps Script has no module system and calls trigger functions by name.

**ESC/POS subsystem** (`src/escpos.ts`, 235 lines). A byte-level command table (`CMD`) covering initialization, alignment, font selection, glyph scaling (`GS ! n` with independent width/height 1–8×), bold/invert/underline, line spacing, feeds, and cut. The `encodeCP437` function maps Unicode block/box/symbol characters to CP437 bytes (e.g., `░→0xB0`, `▀→0xDF`), normalizes curly quotes and dashes to ASCII, drops control characters, and substitutes `?` for anything unmapped. The full protocol is documented in `docs/escpos-protocol.md` (297 lines).

Total codebase: ~1,983 lines of TypeScript + JavaScript + Markdown docs. No dependencies beyond dev tooling (esbuild, clasp, TypeScript, Prettier).

## Key Techniques

### 1. Structured output as creative constraint

The art spec is a JSON schema (`ART_SCHEMA` in `src/art.ts`) with `additionalProperties: false` on every object. The model doesn't emit printer bytes — it emits declarative ops (`{text, width, height, bold, invert, gapless, ...}`) that a deterministic renderer executes. This is the same technique [[OpenAI Structured Outputs]] describes, but applied to a creative rather than efficiency-driven use case. The schema IS the medium: by constraining the output to a 48-column CP437 grid with specific style controls, it forces the model into a creative discipline it wouldn't discover on its own.

The downside: structured output guarantees JSON shape, not field length. In July 2026, a degenerate generation produced valid JSON with a 3,155-character filler title. Because the archive saved it and every later prompt replayed it, the model learned its own loop — within a week the receipt was a page of the model talking to itself. The fix (`validateArtSpec`) checks field lengths against the bounds the schema describes, rejecting a spec before it reaches storage or paper. This is the gap between "valid JSON" and "safe JSON," and it's a failure mode OpenAI's docs don't address.

### 2. The renderer as untrusted-input sandbox

`renderArtSpec` (`src/art.ts:411–463`) treats every op as hostile: sizes are clamped to 1–8, rows are truncated to the column budget (`floor(COLS / width)`), control characters are stripped (they'd be interpreted as printer commands), output is capped at `MAX_ROWS = 150` (~45cm of paper), and `encodeCP437` drops anything CP437 can't print. Full style preludes per op (no state diffing) make output byte-predictable and guard against printer-state leakage between ops. This is the [[Guardrails and Feedback Loops]] thesis applied to a physical output device: deterministic enforcement at render time, not prompt-level pleading.

### 3. Gapless block art via line-spacing calibration

The project's signature visual effect — solid fields of ░▒▓█ characters that tile seamlessly — depends on setting line spacing to exactly one glyph height. The calibration was empirical: `ruler` mode prints rows of ▀ (top-half-black) under candidate `ESC 3 n` values. If n is too low, rows overlap (solid black); too high, white seams appear. Testing n=24, 43, and 48 all rendered identical clean stripes, meaning this firmware clamps spacing up to print-data height — an under-height value can never overlap rows. `ROW_DOTS_A = 24` and `ROW_DOTS_B = 17` are the glyph-height values, and the renderer uses them only for ops marked `gapless: true`. This is documented in `docs/escpos-protocol.md §6`.

### 4. Self-healing rolling context

The `ART_HISTORY` Script Property holds up to 14 recent pieces (title + style note + optional `continues` marker). It feeds back into the prompt so consecutive days differ. But because it's model-written and replayed into model context, a poisoned entry creates a feedback loop. `sanitizeArtHistory` (`src/art.ts:381–394`) validates every entry on read: date must be `yyyy-MM-dd`, title and style must be within bounds, `c` (continuation marker) if present must be a valid date. Invalid entries are silently dropped and the cleaned history is written back — so a poisoned entry is exposed once, not fourteen times. This is the "fix on read" pattern: when storage contains untrusted text that an LLM will re-ingest, sanitize on the way out, not just on the way in.

### 5. Dual-environment harness with exact production mirror

`test-print.mjs` doesn't re-implement the renderer — it loads the built `dist/main.gs` bundle via Node's `vm.createContext`, so `art` mode runs the identical `renderDailyArtReceipt` and `GOLDEN_ART_SPEC` that deploy to Apps Script. `art:live` mirrors the full Anthropic client loop (same `buildArtRequestBody`, same `WEB_SEARCH_TYPES` fallback, same `pause_turn` resumption). The only difference is the `server-side-fallback-2026-06-01` beta header — sent in local testing so a refusal falls back to Opus, but deliberately absent in production so refusals fail loudly to the alert email. This is [[Harness Engineering]] in the purest form: feedforward (the identical bundle guarantees identical output) and feedback (`--dry` previews exact bytes before printing).

### 6. Continuity as in-band memory, not code-side scheduling

Continuations — where a piece deliberately answers an earlier one — are model-judged, not code-rolled. The prompt describes the criteria (holiday after its eve, multi-day event, resonant anniversary), the model sets `continues` to a date when it chooses to continue, and that marker appears in `ART_HISTORY` as `(continues yyyy-MM-dd)`. Future prompts see the marker, which raises the bar for the next continuation. There's no dice, no cooldown timer, no code-side frequency limiter — the model sees the evidence of its own past continuity decisions and regulates itself. The CLAUDE.md is explicit: "that in-band memory is what keeps continuations rare instead of habitual."

## Design Decisions

| Decision | What it optimized for | What it sacrificed |
|---|---|---|
| Google Apps Script (not a proper server) | Zero-cost cloud compute, 6-hour execution limit, no infra to manage | Module system, debugging ergonomics, the whole esbuild+clasp workflow |
| Claude Fable 5 via raw REST (no SDK) | Direct API access from Apps Script's `UrlFetchApp` | Convenience, type safety, SDK-provided retries |
| Structured output (JSON schema) | Guaranteed shape of the art spec | Field length enforcement (patched after the July 2026 incident) |
| Renderer as deterministic byte generator | Safety, predictability, testability | Richness — the model can't invent new rendering techniques |
| CP437 character art (not raster graphics) | Aesthetic constraint, byte-level simplicity | Fidelity — every piece is built from ~200 glyphs designed in 1981 |
| Pi bridge as dumb pipe (`http.server` + `/dev/usb/lp0`) | Simplicity (~40 lines), no printer driver stack | No acknowledgement, no backpressure, no queuing |
| Art history as in-band context (not external memory) | Simplicity, the model sees its own past decisions | Dependence on model discipline (the July 2026 incident) |
| Renderer emits full style preludes per op | Byte-predictable output, no printer-state bugs | Verbose bytestream (7 commands per op before any text) |

The biggest architectural insight: the system is designed so that **any component can fail without data loss, but a print failure is loud**. The `LAST_ART_DATE` guard is only set on success, so an hourly trigger doubles as retry. Failure emails are rate-limited (4-hour window). A poisoned history entry self-heals on the next read. The only unrecoverable failure is a silent print of garbage — and that's prevented by the renderer's clamping, stripping, and capping.

## Comparison Notes

**vs. [[OpenAI Structured Outputs]]**: morningprint is the Anthropic-ecosystem equivalent — `output_config.format` with a JSON schema, not `response_format`. But morningprint surfaces a gap OpenAI's docs don't address: structured output guarantees schema shape, not content bounds. The July 2026 title-poisoning incident (valid JSON, 3,155-char field, model feedback loop) is a failure mode that `validateArtSpec` now catches. The lesson: structured output needs a content-validation layer, not just schema validation.

**vs. [[Guardrails and Feedback Loops]]**: morningprint is a case study in the "linters beat prompts" thesis applied to physical output. The renderer doesn't trust the model's text — it clamps, truncates, strips controls, and caps output. `sanitizeArtHistory` is a read-time guardrail against poisoned context. The self-tightening loop is visible: the July 2026 incident → `validateArtSpec` added → `sanitizeArtHistory` added as defense-in-depth.

**vs. [[Harness Engineering]]**: The `test-print.mjs` / `dist/main.gs` / `art:live` triangle is a textbook feedforward+feedback harness. Feedforward: the local harness loads the exact production bundle, so the preview matches what deploys. Feedback: `--dry` mode shows exact ESC/POS bytes before printing; `ruler` mode empirically validates the column and spacing constants. The `art:live` path mirrors production down to the `pause_turn` resumption loop and web-search-tool fallback — differing only in the refusal beta header, which is a deliberate divergence (local testing handles refusals gracefully, production fails loudly).

**vs. [[Designing Agentic Loops]]**: The daily trigger + 14-day rolling history + continuity exception is a carefully designed agentic loop. Willison's signal — "ugh, I'll have to try many variations" — is inverted here: the point IS variation, and the loop's job is to ensure it (force divergence through history, limit continuity through in-band markers). The loop doesn't converge on a correct answer; it converges on a creative one that differs from yesterday's.

---

*Sources: [[raw/morningprint]], [[summary/morningprint]]*
*Last updated: 2026-08-06*
