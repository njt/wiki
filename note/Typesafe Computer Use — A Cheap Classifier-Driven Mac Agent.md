# Typesafe Computer Use — A Cheap Classifier-Driven Mac Agent

typesafe-computer-use is a macOS agent that drives the computer toward a goal typed in plain English for about a fiftieth of a cent per step. It is counter-programming against frontier-model computer use: instead of shipping a screenshot to a big model each step and waiting seconds for a plan, it reads the screen deterministically (Vision OCR over the frontmost window plus a bounded accessibility-tree walk), hands the resulting numbered list of clickable items to TypeSafe's decision model (Jev) as one multi-`Choice` request, and calls a writing model only when a text field genuinely needs free text. Measured on the same screenshot, one decision costs $0.0002 versus $0.032 for Claude Opus 5 — 155x cheaper, 14–40x faster — and the project is honest about what that buys: every piece of reasoning the frontier model does for free has to be rebuilt as deterministic state. It is also the first working consumer harness in this wiki for the [[System One Models and Jev]] thesis.

---

## Architecture

A single-agent harness, ~2,400 lines of Python, no hierarchy, no LLM in the control loop. The "agent" is a fixed outer loop (`runner.py`) around a perception→decision→action pipeline; intelligence is outsourced to a decision service, and the loop itself is a state machine with stop rules.

Per step (`runner.py:run_step`):

```
capture → OCR (window crop + menu strip) → changed-tile cache →
block merge + goal-echo filter → bounded AX walk → merge sources
into numbered items → one TypeSafe request (3–4 Choices) →
deterministic action → wait → next step
```

Module map (from the README's layout section, verified against the source):

- `macos.py` (480 lines) — the only module touching Quartz, ApplicationServices, or AppleScript. Synthetic input, app activation, window bounds, focused-field lookup, and the bounded accessibility walk. A Linux port replaces this file with xdotool + AT-SPI; nothing else knows the platform.
- `perception.py` (603 lines) — capture with per-round-trip timing, OCR region selection, the changed-tile cache, block merging, the goal-echo filter, AX items, and the ax+ocr merge.
- `decide.py` — state assembly, the three-`Choice` request, confidence composition, the `Noul` type-verification call.
- `writer.py` — the only place free text is generated. Three call sites: `type_text`, `use_browser` with `site: other`, and the once-per-run final answer.
- `actions.py` — one handler per action, each returning a one-line history description that doubles as stall-detection input.
- `runner.py` — stop rules, the run folder, the hand-off of the final screen to the writer.
- `dates.py`, `timing.py`, `report.py`, `cli.py` — date hints, phase stopwatches, annotated screenshots and payload dumps, the `clicker`/`clicker-inspect` CLIs.

Two models, strictly separated. The classifier never generates text; the writer never decides. The writer's three calls each carry a small JSON packet and return a schema-constrained reply (`{fill, text}`, `{ok, url}`, `{achieved, answer}`) via Anthropic structured outputs (`output_config` with a `json_schema` format, `writer.py:_structured`). Only the final answer call sends the capture as a PNG (longest edge 1568 px), and it runs once per run on a stronger model.

## Key techniques

**One request, several mutually exclusive Choices.** `decide.py:decide` sends `kind` (action type), `item` (which numbered item), and `site` (which website) in a single `client.system_one` call, adding a fourth `offscreen` `Choice` only when the accessibility walk found hidden controls. The rule from CONTRIBUTING.md: keep options mutually exclusive — "Every stall found while building this came from two options that meant the same thing," and confidence measures concentration, so overlapping options always read as doubt. The 255-option `Choice` ceiling (`config.py:MAX_OPTIONS`) shapes perception downstream: when items exceed the budget, `kept_by_budget` drops the faintest OCR-only blocks first and never drops an accessibility control for text.

**Confidence composition is asymmetric on purpose.** `Decision.confidence` is `min(kind, item)` for a click and `min(kind, offscreen)` for a press, because a misdirected click lands somewhere wrong and isn't undone. But `use_browser` ignores the `site` answer's confidence entirely (`decide.py` comment and `test_decide.py` cover this): every site outcome is a page the next step can leave, so a split site vote must not stop the run. That distinction — which ambiguities are fatal, which are recoverable — is the sort of judgment a generic agent framework would never make.

**OCR cost engineering: read less, not faster.** Vision is about two-thirds of a step and bills by text volume, so `perception.py` attacks area: crop to the frontmost window with an 8 pt margin, join with a 40 pt menu-bar strip clipped to the window's x-range (the trick that makes the crop pay on full-height windows), clamp to the display. Reuse (`OcrCache`, `ocr_lines`): compare captures at 1/8 scale grayscale (`Image.reduce(8)`), 256 px tiles, a tile counts as changed when mean absolute 8-bit difference exceeds 6.0; changed tiles are union-find-clustered into blobs (side or corner adjacency), each blob becomes a rectangle padded by a tile, and each rectangle is grown until no known line straddles its edge (`grown_for_lines` runs to a fixed point with merging, because a crop through a line returns a fragment). More than four rectangles merge by closest pair; past 60% changed tiles, 60% region area, an app switch, or a window move, the whole region is re-read instead. A line a re-read rectangle touches is dropped and read again whole. Result: `ocr 0.31s (22% of screen, 2 rects)`.

**The bounded accessibility walk** (`macos.py:walk_actionable`): breadth-first with a 4000-node cap, a 0.6 s cap, and a 0.2 s AX messaging timeout, because "frames lie" — Notes reports rows 200 screens down, Chromium parks scrolled-out nodes above the viewport as 1 pt slivers, and a closed `AXMenu` hides thousands of zero-sized items. Pruning rules: skip subtrees wholly off-display (zero-size frames are exempt — containers claim nothing), skip nodes under 4 pt, skip nameless `AXGroup` layout boxes even when pressable, skip `AXMenu` subtrees. Labels resolve `AXTitle` (AppKit) then `AXDescription` (web/Electron), falling back to a short `AXValue`; a bare decorative child borrows its parent's label; list rows take the first shallow `AXStaticText` (two levels, 8 children per level). The walk's children/attrs/actions are callables, so the pruning rules are tested against plain dicts in `tests/test_ax.py` — the platform binding is the only untested part.

**Off-screen controls as a separate action.** `AXPress` does not need visibility: a Notes row thousands of points down, a Chromium link clamped to a sliver, and an auto-hidden Dock's 37 items all take it. The walk keeps the labelled pressable nodes it pruned, deduplicates by role+label, drops labels the visible items already carry, caps at 120, and offers them only as a `press_offscreen` action with its own `Choice` — never mixed into the clickable items, since nothing on the capture points at them and a mouse click would land elsewhere. A refusal is terminal: there is no pixel to fall back on, so it reads as a no-op.

**Source-merged items.** `merge_with_origins` pairs an AX control with the OCR block naming it (intersection over the smaller box ≥ 0.5, plus substring or ≥ 50% shared tokens) into `ax+ocr` items; an `ax` item reads as `button 'Share' (top-right)` in the criteria so the classifier can tell a real control from a line of text. Items are numbered in reading order — rows by median item height, then left-to-right.

**Deterministic reasoning replaces model reasoning.** `dates.py` parses five date formats (month names, ISO, slashed, dash ranges including en/em dashes) and emits "dated 2026-10-13 (in 27 days)" plus "near a line dated …" on neighbours within 60 pt — the README concedes the frontier model "read the event dates off the pixels and compared them unaided." `goal_echoes` drops OCR lines matching the first/last 24 characters of the goal, because the terminal you launched from is on screen and its text is OCR input. `valid_url` admits only clean https URLs with a dotted hostname. CONTRIBUTING.md states the law: "The classifier picks; code decides facts."

**Acting through the tree first.** `click_item` sends `AXPress` to the element the app declared — so the press lands on the control even under a sticky header or cookie banner — and falls back to a synthetic click at the box center. `fill_field` sets the AX value (one message, can't be stolen by a page that moves focus mid-word), reads it back to confirm it took, and falls back to keystrokes; a `Noul` call then scores whether the field holds a sensible value, and under 0.5 the field is cleared. Passwords are never typed — the browser's password manager or an OCR-visible SSO button instead.

**Stopping is designed, not hoped for.** The loop stops on `done`/`none`, confidence under 0.4, two consecutive no-ops (history-line equality plus unchanged URL, plus "refused"/"failed"/"waited" markers in action descriptions), or the step limit; the user can slam the mouse into the top-left 4 px corner from any app (`check_abort` runs every step and during the inter-step delay). Each stop reason is phrased for the writer, which then reads the (possibly re-captured) final screen for an answer grounded in the pixels, told to trust the capture over the OCR text and never to answer from memory.

**Everything is replayable.** Every run writes `runs/<timestamp>/`: raw and annotated captures (blue OCR, orange AX, red chosen, green focused field), the exact state and criteria sent, every probability returned, per-phase timing. `--image` replays a saved capture offline — always full-frame, never using the tile cache, so a repro matches the original run. CONTRIBUTING.md asks for bug reports with the run folder attached.

## Design decisions

- **Optimized for cost per step and auditability; sacrificed generality and breadth.** Single display only; it fights the user for mouse and focus during `--act`; the site catalog is eight URLs and deliberately small (the writer covers the rest). The bet is that most steps in most GUI flows are one choice from a short list, not a plan.
- **The classifier does no arithmetic.** Calendar math, URL validity, focus state — all computed in code and fed in as state. This is the direct inversion of the frontier-agent pattern, and the README's caveat quantifies it: the honest benchmark row is the one admitting Opus needed no date parser.
- **Perception is never trusted alone.** AX coverage is measured per app and published: Finder 100% of on-screen controls labelled, Chrome 88%, Slack 85%, Notion 68%, Spotify 0% (its CEF shell exposes three window buttons). So AX is "a bonus source, never a replacement," merged with OCR rather than replacing it.
- **Tests target the pure logic** — dates, block merging, reading order, echo filter, decisions, the tree walk against fake trees, the OCR cache — ~1,400 test lines for ~2,400 source lines, CI on macOS runners. The platform bindings are the accepted untested surface.
- **Weaknesses worth naming:** the whole thing is macOS-only today; two identical labels get only a coarse 3×3 region hint and split the vote; the classifier is a vendor API (a `TYPESAFE_API_KEY` outage stops every decision); and the cost comparison is self-reported, on one screenshot, one decision each — with the date-parsing crutch disclosed, which is to the authors' credit.

## Comparison notes

- [[System One Models and Jev]] argued reliable automation decomposes into many small calibrated judgments whose probabilities, not just argmax, feed branching logic. This repo is that thesis compiled into a working harness: `kind`/`item`/`site` `Choice`s gate the loop, and a `Noul` check verifies the post-condition of every typed field. Where the blog argued the interface, this shows the interface in production.
- [[You Could Have Built Jev]] concluded a Jev-like is just restricted-softmax logits over two token IDs — nothing mysterious. This project is the retort to the implied "so what": the logits are the easy part, and the actual engineering is the deterministic perception, state assembly, and mutually exclusive option sets around them.
- [[Computer Use is 45x More Expensive Than Structured APIs]] attacked pixel-driven agents by leaving pixels for structured APIs; its structural claim was that step count is set by the interface. This repo accepts the pixels-and-steps world and instead collapses per-step cost ~150x — a middle path for apps you don't control — though Reflex's silent-failure critique still bites here, which is why `none` and no-op stalls get explicit stop reasons.
- [[Cua-S1 — Small Specialist Computer-Use Models]] is the same "small models for computer use" thesis by opposite means: Cua-S1 trains a specialist vision model with snapshot-bound element tokens and a dry-run default; typesafe-computer-use trains nothing at all, replacing the specialist model with deterministic perception plus a general decision model. Both refuse to let a frontier model improvise a plan per step. For the maximal-stacks comparison, [[Cua — Computer Use Agent Platform]] runs the VLM as the decision-maker inside a 245K-line orchestration layer — the exact architecture this repo bets against.

One adjacent data point: [[xa11y — Desktop Automation via Accessibility APIs]] pitches the accessibility tree as a full replacement for screenshot vision; this repo's measured per-app AX coverage (Spotify 0%, Notion 68%) is concrete evidence for xa11y's weaker claim — AX is robust where it exists, and unevenly distributed.

Tags: #tool #project #agents #computer-use #macos #ocr

---
*Sources: [[raw/typesafe-computer-use]], [[summary/typesafe-computer-use]]*
*Last updated: 2026-09-22*
