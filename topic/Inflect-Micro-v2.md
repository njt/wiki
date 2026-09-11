# Inflect-Micro-v2

A 9.4M-parameter end-to-end text-to-speech model by Owen Song that generates 24 kHz mono audio from English text on CPU or CUDA. Apache-2.0 licensed, deterministic-seed reproducible, and small enough (37.5 MB FP32) that the entire TTS pipeline — phonemization, alignment, stochastic synthesis, waveform decoding — fits comfortably on-device with no cloud dependency.

---

## What It Is

Inflect-Micro-v2 sits in the VITS family of end-to-end text-to-waveform generators. It chains an eSpeak-ng English phoneme frontend through monotonic alignment, a 192-channel latent space with 4 residual coupling flow blocks, and an integrated alias-reduced neural waveform decoder that outputs 24 kHz mono. The whole thing is 9,356,513 parameters — genuinely under 10M — and ships as a single 37.53 MB `.pth` file with a self-contained inference CLI.

There's also an ONNX runtime export (separate repo, CPU/CUDA/DirectML, no PyTorch import needed) and a smaller sibling, Inflect-Nano-v2, at 3.97M parameters / 15.97 MB for footprint-constrained deployments.

The model supports three runtime controls: `speed` (0.5–2.0×), `variation` (0.0–1.0, lower = steadier), and `seed` (integer, for reproducibility). Long text is handled via punctuation-aware chunking with controlled pause insertion and edge fades — a practical heuristic rather than a native long-form architecture.

## Key Quotes

> "Complete local text-to-waveform speech synthesis under 10M parameters."

This is the mission statement, and it's delivered. The entire pipeline — frontend through decoder — stays under 10M. For context, [[Pocket TTS]] (Kyutai) claims 100M parameters for a similar local-TTS pitch. Inflect is an order of magnitude smaller and still produces competitive 24 kHz output.

> "No single metric is treated as proof of overall superiority."

Rare honesty in a model card. Owen Song benchmarks against KittenTTS Nano, Piper Low, and Supertonic 3 across human preference (66.2% win rate in a blind community study), UTMOS22 predicted naturalness (4.395), and multi-ASR word error rate (3.99% headline). Each metric tells a different story — Inflect leads on naturalness but trails the pack on raw CPU throughput (6.28× real-time vs. Piper's 31.37×). This is what honest benchmarking looks like.

> "The training corpus-generation pipeline and private filtering infrastructure are not part of the public release."

The voice is synthetic, not a real person's — but the synthesis pipeline that produced the training data is closed. This is a common tension in open-weight TTS: the weights are open, but the *data provenance* is a black box. You can run the model, but you can't reproduce how it was made.

> "Unfamiliar phrasing can become flatter, less expressive, or less stable."

An honest limitation statement that names a real failure mode. Single-voice TTS models overfit to their training distribution; throw unusual syntax, technical jargon, or foreign names at them and the latent synthesis wobbles. This isn't a bug — it's a structural property of compact fixed-voice models.

## Key Themes

#tool #text-to-speech #local-inference #open-source #speech-synthesis #compact-models

## Architecture Notes

**VITS with an integrated decoder.** The VITS family (Conditional Variational Autoencoder + Normalizing Flows + GAN-based HiFi-GAN decoder) combines the three traditional TTS stages (text→spectrogram→waveform) into a single end-to-end model. Inflect v2's decoder is "alias-reduced" — it uses anti-aliasing filters in the upsampling layers, which matters for the 24 kHz output band. This isn't novel architecture; it's careful engineering of a known design.

**Monotonic alignment, not attention.** The model uses monotonic alignment search (MAS) rather than learned attention for duration prediction. MAS guarantees the alignment is actually monotonic — each input phoneme maps to a contiguous output segment, no crossing. This is the right choice for a fixed-voice model: attention can learn exotic alignments that sound fine in training but produce garbled output on unseen text. Determinism over flexibility.

**Punctuation-aware chunking as the long-form strategy.** Rather than training an autoregressive model that can synthesize arbitrary-length audio, Inflect chunks at punctuation boundaries, synthesizes each independently, and stitches them with fades. This is pragmatic but creates audible seams at transitions — a tradeoff the model card acknowledges honestly.

## Critical Analysis

**The compact-TTS space is getting genuinely interesting.** Pocket TTS at 100M parameters, Inflect-Micro-v2 at 9.4M, Inflect-Nano-v2 at 4M. These aren't incremental improvements over the previous generation of local TTS (Piper, eSpeak's own synthesis) — they're a different category of model entirely. A 37.5 MB file producing 24 kHz audio at 6× real-time on CPU means TTS is now a solved problem for any deployment that doesn't need voice cloning or multi-speaker support.

**The benchmark story is more interesting than a single number.** Inflect wins on human preference and naturalness but loses on raw speed. Piper is 5× faster but sounds worse. The right model for your use case depends entirely on whether you're optimizing for quality or throughput. The model card's willingness to present the full comparative picture rather than cherry-picking a single "we win" metric is a model for how open-source releases should communicate performance.

**The missing ingredient: voice diversity.** A single fixed English male voice is a ceiling, not just a limitation. The most compelling use cases for local TTS — personalized assistants, accessibility tools, language learning — want voice variety. The experimental fine-tuning toolkit that *replaces* the voice rather than adding to it is an honest but limiting design choice. Voice cloning à la [[Pocket TTS]] is the feature that would make this model a platform rather than a component.

**The synthetic-voice provenance problem is structural.** Owen Song is transparent that the voice is synthetic — it doesn't belong to a real person — but the pipeline that created it is closed. This is better than training on scraped audio of unknowing speakers (the Suno approach, per [[Suno Training Data Breach]]), but it means the model is open-weight, not open-data. For the local TTS space to mature, someone needs to release a fully-reproducible training pipeline: open data → open phonemization → open synthesis → open weights. Inflect is two-thirds of the way there.

**The ONNX export is the sleeper feature.** PyTorch is heavy. The ONNX runtime export (CPU/CUDA/DirectML, zero PyTorch imports) makes this model embeddable in applications without dragging in the entire PyTorch ecosystem. For production deployments — embedded devices, mobile apps, desktop software bundling TTS — the ONNX path is the real product, and the PyTorch checkpoint is the development artifact.

## Comparison to the Wiki's Existing Voice Coverage

[[Pocket TTS]] (Kyutai, 100M params) claims CPU-only operation with voice cloning. Inflect operates at one-tenth the parameter count with a fixed voice. They're complementary: Inflect for "I need a known voice, small footprint, deterministic output"; Pocket TTS for "I need to clone any voice, don't care about model size."

[[Local and Open Source Inference]] declares "Text-to-speech is there: Pocket TTS runs voice cloning on CPU at 100M parameters." Inflect-Micro-v2 pushes that claim further: TTS is not just "there" at 100M params — it's there at under 10M, with competitive quality, and the smaller model opens deployment targets that Pocket TTS can't reach.

[[Building Production-Ready Voice Agents]] identifies TTS latency as one component of the voice agent latency budget (100–200ms per turn). At 6.28× real-time on CPU, Inflect produces one second of audio in ~159ms — right in the sweet spot for interactive voice agents where the P95 budget is 1.5–2.5 seconds end-to-end.

[[MiniMax Models]] covers a commercial speech API with 40 languages; Inflect is the open-source counterpoint — one language, one voice, zero API calls.

[[Piano Autocomplete — On-Device Music Copilot]] is the same compact-model-on-device move in a different modality: a 125M-parameter transformer doing next-note prediction at ~108 notes/sec on an iPhone 15. Inflect's 9.4M-param TTS and that 125M-param piano model are two data points in an emerging genre — a complete, useful model per modality, small enough to run with no cloud.

[[AuK — Speech Foundation Model]] is the foundation-model counterweight at the other end of the same VITS-descended family: 1.5B params, zero-shot voice cloning and instruction-based editing via a frozen Qwen2.5-Omni encoder, but it needs a GPU and pays a full LLM forward pass per utterance. Inflect's "smallest possible single-voice TTS" and AuK's "one model for every speech task" are complementary, not competing — different questions, same lineage.

---

*Sources: [[raw/inflect-micro-v2]]*
*Last updated: 2026-07-29*
