# Whistle — On-Device Speech Recognition

Cactus Compute's Whistle is a 16.9 MB speech recognition model that transcribes, timestamps words, and emits speech embeddings entirely on-device — CPU-only, no dependencies, seven European languages, 30-second clips. It shares a C++ inference engine (Needle) and model architecture with their text models, and ships prebuilt for 17 platforms from watches to microcontrollers to the browser.

---

## What It Does

Three jobs in one model, all local:

- **Transcription** — 16 kHz mono audio, up to 30 s per pass, in English, German, French, Spanish, Italian, Dutch and Polish, with automatic language detection emitted as a vocabulary token rather than returned out of band.
- **Word timestamps** — every word with start, end and probability, aligned from decoder attention.
- **Speech embedding** — one encoder row per 80 ms frame, no transcript needed.

## Architecture Notes

The design philosophy is *one engine, many modalities*. Whistle is not a bespoke ASR stack; it is Needle's language-model block list with speech grafted on:

- Encoder: 8 Simple Attention blocks — mHC residual lanes and Monarch Hadamard MLPs instead of ordinary feed-forward nets. Non-causal: a frame at 3 s attends to one at 12 s.
- Decoder: 8 laddered blocks at width 512, 8 query heads to 2 KV heads, engram lookups at layers 3 and 7 over 18,432 slots.
- The single speech-specific addition is a **gated cross attention per decoder layer**, with K/V projections computed once when the clip arrives and held for the whole decode. This is the clever bit: 5-beam search costs five short transcript caches, not five passes over the audio.
- Keyword biasing walks an Aho-Corasick automaton alongside the beams, lifting the log probability of user-supplied phrases as they match — a neat deterministic mechanism for wake-words and names the acoustic model habitually fumbles.
- The ladder is trained: every depth from 2 layers up is a model in its own right, selected at load time via `--audio-depth`. The encoder is never sliced.

## Benchmarks, Honestly Framed

Whistle beats Whisper base on LibriSpeech test-clean/test-other, SPGISpeech, Earnings-22 and FLEURS average; Whisper wins TED-LIUM, AMI and MLS — while being 145.3 MB to Whistle's 16.9. Latency is where the small model shines: no 30-second padding, so time to first token tracks clip length (5.9 ms at 5 s vs 36.3 ms at 30 s). The authors are refreshingly explicit about methodology: 86,174 utterances scored with the Whisper normalizers, published figures used for competitors, and a no-test-leak claim verified by audio checksums and speaker IDs across every test set.

## Deployment Is the Argument

The real thesis is the packaging: prebuilt engines for 17 targets (macOS to Android, watchOS, RISC-V, MIPS, WASI), a three-function C API (`needle_load`, `needle_transcribe`, `needle_embed`), no environment variables, and a `.cact` container that lets the same binary do speech, text, or both — including transcribe-then-tool-call in one pass, returning one JSON object with `audio_`-prefixed speech fields alongside the tool calls.

---

## Key Quotes

> "It is one 16.9 MB file, runs on the CPU with no dependencies."

The whole pitch in one sentence. At this size, speech recognition stops being a service and becomes an asset — something you ship inside a firmware image.

> "Those projections run once when the clip arrives, 375 frames across 8 layers, and are then held for the whole decode. Five beams therefore cost five short transcript caches, not five passes over the audio."

The encoder-decoder interface is engineered for beam search economics, not just accuracy. Most ASR write-ups never talk about this; it shows the team is optimizing the whole system, not the leaderboard.

> "Keyword biasing walks an Aho-Corasick automaton over the phrases you pass in, alongside the beams, and lifts their log probability as the automaton advances."

Constrained decoding done with a 1970s string-matching algorithm instead of more model. Very much in the spirit of [[When Smaller Models Win]] — the deterministic tool where deterministic tools win.

> "The engine reads no environment variables. Every behaviour is a compiled default or an explicit flag."

A deployment philosophy stated as an API contract. For embedded targets this is not a preference; it is the difference between shippable and not.

## Key Themes

#concept #tool #pattern — edge inference · small models · multi-modality via shared architecture · deterministic decoding constraints · benchmark transparency

## Opinionated Take

This is a quietly excellent release note — arguably a model of what model release posts should be. The benchmark section names its winners and losers (Whisper takes three datasets), states its normalization and leakage-verification methods, and doesn't hide behind averages. The architecture section reads like it was written by someone who expects the reader to implement it.

The strategic claim underneath is more interesting than the model: by making speech and text load into the same engine from the same container, Cactus is arguing that modalities should be a loading decision, not an architecture decision. Whether the Monarch-Hadamard-plus-engram block list is actually good (rather than merely compact) won't be settled by seven benchmarks, but the packaging bet — one binary, 17 targets, no env vars — is exactly right for the wearables/robotics/microcontroller market where every competitor is busy shipping a cloud round-trip.

Worth watching: the transcribe-then-tool-call path (`needle_complete` taking audio directly and returning tool calls plus speech fields) is the first sign of a voice-native agent loop that never materializes a transcript in caller code.

## Related Pages

- [[Inflect-Micro-v2]] — the mirror-image release from the same niche: a 9.4M-parameter on-device *text-to-speech* model. Whistle (speech in) plus Inflect (speech out) sketch a complete local voice stack under 60 MB; Whistle strengthens that page's thesis that sub-40 MB speech models are a real product category.
- [[Indexing 669 GB of GoPro Videos with Local ML]] — Whisper there cost 25 hours of M1 Max time for 15 hours of audio; Whistle's clip-length-tracking latency and CPU-only runtime complicate the assumption that transcription is the expensive stage you must batch offline.
- [[When Smaller Models Win]] — Whistle is a concrete instance of the essay's argument: a narrow, well-scoped task where the small self-hosted model beats the bigger general one on the axes that matter (latency, footprint, deployment surface).
- [[Bonsai 27B]] — the other end of the same size ladder: PrismML's quantization pushing 27B-class LLMs onto phones while Cactus pushes speech down to microcontrollers. Together they map how far down the device hierarchy "on-device" now reaches.

---
*Sources: [[raw/whistle]], [[summary/whistle]]*
*Last updated: 2026-10-03*
