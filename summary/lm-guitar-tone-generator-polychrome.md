---
url: https://vishsubramanian.me/lm-guitar-tone-generator-polychrome/
title: "Dialing In the Ghost in the Machine: LLMs for guitar tones"
author: Vishwanath Subramanian
date_fetched: 2026-06-15
date_published: 2026-05-31
categories: Code, AI × Music
github: https://github.com/vishwanath79/tonellm
topics:
  - agent-architecture
---

# Dialing In the Ghost in the Machine: LLMs for guitar tones

Author: Vishwanath Subramanian
Date: May 31, 2026
GitHub repo: tonellm

## Core Concept

Tone LLM (`tonellm`) is a Python tool that converts a text description of a desired guitar tone — optionally paired with a reference audio file — into a `.pdpreset` compatible with Polychrome DSP (the McRocklin Suite amp simulator). The key architectural insight: the LLM never writes plugin XML directly. Instead, it fills a small JSON contract (a Pydantic model), and deterministic Python code handles the XML translation.

The author describes being "humbled pitting my limited pre-existing tones on amp sims versus the settings an LLM can spit out which sound devastatingly better."

## Architecture (5 Layers)

### Layer 1: The Contract (`ToneDescriptor`)

A Pydantic model with normalized knob values (0.0–1.0), enums for cab/boost/delay/reverb, and human-facing fields:
- `confidence` — the model's self-rating of its output
- `production_notes` — pedagogical explanation of *why* the settings work
- `daw_recommendations` — suggestions for what the plugin can't do (double-tracking, bus compression)

The system prompt explicitly forbids the model from telling users to add delay in the DAW when Polychrome already handles delay internally.

Users can edit the sidecar JSON and re-run translation without another API call:
```
tonellm from-descriptor RunRiot.tone.json \
  -out "/Users/Shared/PolyChrome DSP/Presets/McRocklin Suite/User/RunRiot_v2.pdpreset"
```

### Layer 2: The System Prompt (tone_system.md)

Domain knowledge lives in a markdown file loaded as the LLM system prompt — described as "prompt as curriculum" rather than fine-tuning. It contains:

**Era cheat table** — mapping styles to amp/cab combinations:
- 80s hair/arena (Hysteria, Pyromania): JCM800 + Rockman layering, greenback/G12-65 cab, mids forward/not scooped, compressed, subtle chorus, plate verb
- 80s thrash (Master of Puppets era): Marshall + boost, greenback, tight and mid-forward
- Modern djent/prog: High-gain modern with V30 cab, tight low-cut, present mids

**Myths the model is told to avoid:**
- "Heavy = scooped mids" is "false for most pre-1995 rock and much modern prog" — default mid is 0.5–0.65
- Gain past ~0.7 often mushes pick attack; prefer 0.55–0.7 plus a screamer boost
- Stereo widener on rhythm defaults to off; double-track in the DAW instead

**Guitar compensations** — RG3550 hot ceramics get slightly reduced cab air and increased cab low-cut; active EMGs get gain pulled back; drop tunings tighten low end in cab filters.

### Layer 3: Measuring the Recording

An optional `-ref` flag runs librosa on the first ~30 seconds (or a `-section` window) extracting interpretable features:
- Spectral centroid/rolloff → brightness
- Spectral flatness → tonal vs noise-like (distortion pushes it up)
- Crest factor → squashed vs dynamic
- Band energy (60–250 Hz, 250–2k Hz, 2–8 kHz)
- Tempo estimate

These become a natural-language paragraph appended to the user prompt. The function `summarize_for_llm()` explicitly tells the model to "Ground the descriptor in these measurements" and, on full mixes, "down-weight low-band energy and crest factor when inferring the guitar's character."

The audio analysis flavors the descriptor but doesn't replace the text query — era, style, and player identity still come from the user's words.

### Layer 4: The LLM Step

The author experimented with agent tool loops and ReAct chains but found them "a bit muddy and overcomplicated." The final design is a single `client.chat()` call returning one JSON object, providing predictable cost, failure, and debugging.

**System vs user prompt split:**
- System message (tone_system.md): the curriculum — rarely changed
- User message (`_build_user_prompt`): the task — changes every run

The author tested multiple LLMs and found "deepseek-v4-pro seemed to give me the closest results."

**Four-layer defense for structured output:**

| Layer | Mechanism | What it catches |
|-------|-----------|-----------------|
| 1 | Ollama `format=ToneDescriptor.model_json_schema()` | Wrong types, missing fields at generation time |
| 2 | `_extract_json()` | Markdown fences, prose before/after JSON |
| 3 | `ToneDescriptor.model_validate_json()` | Enum typos, out-of-range floats |
| 4 | `polychrome.py` | Assumes valid descriptor, maps deterministically |

**Temperature 0.4** keeps repeated runs similar while allowing interpretive choices from the era table.

**The contract/adapter pattern** means:
- Swap LLM vendors without touching XML
- Swap output targets (Neural DSP, Helix, etc.) without retraining
- Test the adapter with hand-written JSON — no API key needed

The `.tone.json` sidecar is described as "the human-editable layer of the contract. The LLM is a *proposal generator*; you are the approver."

**Failure mode table:**

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Pydantic error on `amp_type` | Model used synonym not in schema | Tighten system prompt field names |
| JSON inside fences only | Normal for hosted models | `_extract_json` handles it |
| Tone sounds wrong but validates | Reasoning error, not syntax error | Edit sidecar; read production notes |
| Reference audio misleads EQ | Full mix, no `-section` | Isolate solo; trust text query |

**General recipe** for LLM → proprietary config (4 steps): define a small schema; put domain rules in system prompt; put variable evidence in user prompt; generate with schema constraint + validate + deterministic codegen with an editable artifact.

### Layer 5: Deterministic Translator

`polychrome.py` loads `templates/base.pdpreset` and patches only what the descriptor implies. Unmentioned attributes inherit from the template, providing forward compatibility.

Cab archetypes map to internal IR slot integers (starting guesses refined by ear) — e.g., `greenback_4x12` maps to slot 1, `g12_65_4x12` to slot 2, `v30_4x12` to slot 4.

Amp channel selects both `AmpSel` and the knob prefix (`GA` for gain, `CA` for clean, `CB` for crunch). Delay "roles" (`quarter`, `dotted_eighth`, `ambient`) map to sync subdivision positions.

This layer works identically whether the descriptor came from the LLM, a hand-edited sidecar, or any future non-LLM source.

## How It's Run

**Web UI** (Streamlit, local):
```
pip install -e ".[ui]"
tonellm ui
```
Launches on `localhost:8501` with a tone request field, optional reference upload, section range, and download for preset + sidecar.

**CLI — text only:**
```
tonellm tone "Phil Collen Run Riot rhythm" \
  -guitar rg3550 -tuning E \
  -out "/Users/Shared/PolyChrome DSP/Presets/McRocklin Suite/User/RunRiot.pdpreset"
```

**Environment** (`.env`):
```
OLLAMA_API_KEY=your_key_here
OLLAMA_HOST=https://ollama.com
OLLAMA_MODEL=kimi2.6
```

Both CLI and UI call `run_tone()` in `service.py` — one pipeline, two interfaces.

## Honest Limits

1. Full mixes lie about guitar character unless you isolate a solo section
2. Cab slot mappings are calibrated by ear, not absolute ground truth
3. Presets are starting points, not forensic clones
4. Not a replacement for playing — room, pick, doubles, and attitude still matter
5. "Tone is in the fingers"

The author frames the project not as "Mutt Lange in a box" but as a repeatable pipeline that reclaims time spent tweaking knobs for actual playing, calling it "deeply satisfying to light up the room with the tone that shredded arenas in the glory days."
