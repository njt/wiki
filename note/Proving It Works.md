# Proving It Works

A Claude Code plugin from Prime Radiant that records a narrated movie proving software actually runs — and, more importantly, gates that movie with a mechanical checker that catches the silent defects (frozen picture, narration over a dead screen, dropped words, missing subtitles) that per-frame inspection structurally cannot see. Its thesis is that a demo video is evidence, and evidence that quietly lies is worse than no evidence.

---

## Architecture

A single plugin, `proving-it-works`, that ships one skill directory — `skills/proving-it-works-with-a-movie/` — containing a `SKILL.md` router, six reference documents (one per route plus narrating and assembling), and five Python scripts. The pipeline is a straight chain over one data file:

```
scenes.yaml → narrate → assemble → make-subtitles → burn-subtitles → check-movie
```

`scenes.yaml` is the load-bearing abstraction. It is read by both `narrate` (to render one voice clip per scene) and `assemble` (to build the cut), so narration and picture stay in sync *by construction* — each scene is a `max(narration, visuals)` segment, the short half padded (`tpad=stop_mode=clone` freezes the last frame, `apad` pads silence). `narrate` writes `manifest.json` with the exact text and ffprobe-measured duration of each clip; `make-subtitles` reads that manifest and the scene offsets `assemble` emits (`segments/offsets.json`), so no downstream step ever guesses a timing.

Distribution is multi-harness: `plugin.json`/`package.json` plus per-harness manifests (`.claude-plugin`, `.cursor-plugin`, `.codex-plugin`, `.devin-plugin`, `.hermes-plugin`, `.kimi-plugin`, `.opencode`, `.pi`, `.agents`) and an `everyharness.yaml`, each pointing at the same `skills/` tree. The e2e demo (`examples/e2e/`) is the plugin proving *itself*: a clean container installs the plugin from the public marketplace, an agent inside it uses the skill to make a narrated movie, and `check-movie` verifies that movie.

## Key techniques

**The mechanical gate (`scripts/check-movie`).** This is the interesting part. It samples picture and sound on one timeline and compares them: video is decoded at `fps=1,scale=320:-1`, each frame greyscaled, and a "new state" counts when >0.2% of pixels shift by more than a per-pixel grey delta of 8; audio is decoded to 8 kHz mono PCM and windowed into per-second RMS in dBFS, with −45 dB marking "someone is talking." The gate fails when the last visible change lands before 40% of runtime while narration continues >5 s past it (the exact failure that motivated the plugin), when the picture never changes, when the audio is silent, or when a narrated movie's subtitles are missing or end before the speech does. It always writes a `contact-sheet.png` — twelve sampled frames laid out as a grid — and then tells you the part no script can do: *open it and look*. The known blind spot is written into the docstring: 1 Hz sampling means any beat shorter than a second falls between samples.

**The verbatim narration gate (`scripts/narrate`).** Narration is where the most embarrassing silent failures live, so `narrate` renders each clip and then listens back to it with a local ASR (faster-whisper, in its own `uv` env so the script stays light). The non-obvious choice is the drift metric: it flags *missing or invented content* — a length change >15% or a run of ≥4 consecutive changed words — rather than exact word match. The rationale (in `narrating.md` and `structural_drift`'s docstring) is subtle and correct: a small ASR mangles unusual names, but a *dropped* word scores as *more* similar than two mispronounced ones, so a strict ratio passes the real defect and fails the harmless one. Engine selection is automatic — OpenAI's deterministic `/v1/audio/speech` when a key exists, local Piper when it doesn't — so a container with no secrets can still narrate; a chat-model voice (`gpt-audio-1.5`) is available but gated, because it ad-libs preambles ("Sure, here it is:").

**The silent-failure catalogue.** The route docs are dense with hard-won traps, each with a code-level fix: headless Chrome renders ttyd's `<canvas>` blank without software GL (`--use-gl=angle --use-angle=swiftshader`); `tmux send-keys` into a still-running program lands in *its* stdin; browser automation draws no cursor so clicks look haunted (inject an animated ring overlay, type at ~55 ms/char); `ffmpeg` eats a loop's stdin without `-nostdin`; Homebrew's ffmpeg lacks libass so `burn-subtitles` probes `-filters` and falls back to a soft track rather than silently shipping an unburned movie; `Page.captureScreenshot` during a navigation never returns, so every capture is raced against a ~700 ms timeout.

## Design decisions

**Optimize for honesty, not convenience.** The skill's hard rule is *never mock, stage, or reenact*: if a beat can't be shown for real, cut it and say why. Recording must happen against a copy of the data, never the live tree. When OS screen capture is permission-blocked, the documented pivot to a log-rendered reel is "the honest outcome, not a fallback to apologize for."

**The gate is deliberately necessary-but-not-sufficient.** `check-movie` exits nonzero with "NOT SHIPPABLE" on the egregious cases, but its thresholds are heuristics tuned against real good and bad movies, and its own output insists it cannot tell you a movie is *right* — the contact sheet and human eyes are the final layer. This is a clean division of labor: the deterministic script catches what automation can, and hands the residue to the one sensor that can judge content.

**Measured, never guessed.** Every timing in the pipeline comes from ffprobe or the manifest, down to subtitle cues being apportioned by character count within a scene's measured audio. The README's framing — "per-frame verification cannot see a defect that lives *between* frames" — is the design's root.

## Comparison notes

- **[[What You NEED to Know Before Touching a Video File]]** — that guide's craft ("use your eyes," quality as fidelity-to-source) is exactly what `check-movie`'s contact sheet operationalizes into a scripted gate. Proving It Works is the guide's principles turned into a pipeline with an enforcement step.
- **[[Guardrails and Feedback Loops]]** — `check-movie` is a deterministic computational sensor applied to a generated artifact that is *not code*: a movie. It extends the "linters beat prompts" thesis one domain out — the frozen-picture failure it catches is precisely the reward-hacking pattern (artifact satisfies every check but is broken) that synthesis documents for code.
- **[[Claude Code Skills System]]** — a worked instance of the skills pattern: `SKILL.md` as a thin router, four route documents loaded on demand as progressive disclosure, and five bundled scripts the skill executes to sidestep the permission-prompt loop.
- **[[Prime Radiant (Company)]]** — another Prime Radiant spillover tool from Jesse Vincent, but MIT (not the Apache 2.0 of the CLI trio) and packaged as a cross-harness plugin rather than a standalone CLI. It sits in the same niche as the company's "agents write, humans review" pipeline: here the human review is watching the contact sheet.

---
*Sources: [[raw/proving-it-works]], [[summary/proving-it-works]]*
*Last updated: 2026-08-14*
