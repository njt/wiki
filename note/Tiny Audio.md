# Tiny Audio

Tiny Audio (Alex Kroman) is an open-source speech-to-text system you can train for $25: a frozen pretrained speech encoder joined to a pretrained LLM by a small trainable MLP projector, hitting 1.84% WER on LibriSpeech test-clean and 7.42% pooled across 12 benchmarks while training only ~80M parameters. The repo is as much pedagogy as product — a free 3.5-hour course, a 5-minute laptop smoke test, and config comments that read like an engineering lab notebook. It matters because it is the most fully documented worked example of the "adapter, not model" school of building multimodal systems.

Tags: #tool #project #asr #lora #local-inference

---

## Architecture

The core is ~4,900 lines of Python in `tiny_audio/`, structured as a standard transformers-compatible model plus an unusually complete ops shell:

- **`tiny_audio/asr_modeling.py`** (2,191 lines) — `ASRModel(PreTrainedModel, GenerationMixin)` composes three swappable pieces loaded from config: audio encoder, projector, LLM decoder. Each is independently replaceable (`audio_model_id`, `text_model_id`); LoRA setup, dtype resolution, and EOS/turn-end token derivation all live here.
- **`tiny_audio/projectors.py`** — the only part trained from scratch in the published recipe: RMSNorm → Linear(encoder_dim·k → hidden) → GELU → Linear → llm_dim, with frame stacking (concat k=4 adjacent frames) as the downsampler. A registry (`PROJECTOR_CLASSES`) keeps it extensible.
- **`tiny_audio/asr_pipeline.py`** (751 lines) — chunking, batching, timestamp/diarization orchestration for the `transformers` pipeline entry point.
- **`tiny_audio/diarization.py`** — Nemotron-3 speaker frame classification; each speaker's speech is transcribed on a copy of the audio where everyone else is zeroed out (see `masked_audio`), a zero-shot port of NeMo's `masked_asr` recipe.
- **`tiny_audio/alignment.py`** — word timestamps via Qwen3-ForcedAligner-0.6B, chunked at 5 minutes per forward pass, preserving original punctuation/casing through the alignment split.
- **`scripts/serve.py`** — a three-role threading model: the Starlette event loop only accepts requests; per-request worker threads do CPU work (ffmpeg decode, chunking, features); a single GPU thread owns the model and drains a queue into length-near batches around the oldest chunk. Chunks queue during a batch, so the next batch fills itself under load.
- **`scripts/eval/` + `ta runpod`** — benchmark harness that scores the model *and* commercial APIs (AssemblyAI, Deepgram, ElevenLabs, Apple Speech) on the same samples, plus pod provisioning/VRAM planning.

Config is Hydra: every experiment recipe is a YAML file, every knob an override (`+experiments=stage_1`, `model.projector_hidden_dim=2048`).

## Key techniques

The config files are the second half of the codebase — the comments in `configs/experiments/granite_qwen_frozen.yaml` carry measured findings most papers never state:

- **Projector norms are a trap.** A trailing RMSNorm pins *every token* to the same magnitude, but a trained projector's per-token RMS spread (CV 0.373, max/min 9.4x) is the signal — silence vs speech, confident vs ambiguous frames. Adding one destroyed training: loss 3.166 vs 0.319 at step 300. Similarly, an intermediate norm makes the feeding linear scale-invariant, so its weight inflates without bound while silently decaying its effective learning rate (angular step ∝ lr/‖W‖).
- **LoRA can perturb harder than full fine-tuning.** Measuring ‖dW‖_F/‖W‖_F on a shipped r=64, alpha=128 adapter: 18.39% mean across 186 matrices — versus 13.31% for *fully fine-tuning the same decoder*. Alpha/r=1.0 halves that. Rank stayed 64 because the singular spectrum said rank 32 would clip over half the modules.
- **Unfrozen encoders random-walk under weight decay.** Displacement vectors of the top-4 encoder blocks were orthogonal (cos 0.118→0.004) and sub-diffusive — AdamW's decay pulled them to an "equilibrium radius" where they stopped learning. And you cannot undo it afterwards: reverting those blocks to base weights cost 14–16 WER points, not because the drift learned anything but because the projector and LoRA *co-adapted* to the random endpoint.
- **Dropout math made visible.** Elementwise dropout at p injects sqrt(p/(1−p)) relative RMS noise — p=0.05 puts 22.9% noise on every audio token, well past NEFTune's 5–15% band, on the one module with no alternate path.
- **Scale drift that doesn't matter.** The projector's output settled at ~48–66x the decoder's embedding RMS; measured dL/d(log c) was +0.0005 ± 0.0273 — a drift most engineers would "fix" that has no cost. The comment names the real levers: projector LR or weight decay, not clamping.

## Design decisions

- **Frozen everything except the adapter.** With the encoder frozen, backward never enters it and its BatchNorm modules stay in eval mode (a train-mode slip once cost 25 WER points on Earnings22); fp32 encoder weights become pointless since there are no updates to lose, freeing ~1 GB for batch size.
- **One epoch, more steps.** Rather than a second data pass, gradient accumulation drops 2→1 so the identical forward/backward budget yields ~2x optimizer steps — with the failure mode documented (`num_train_epochs: 2` replays the same 600K subsets because the loader's resampling happens once).
- **Sample-matched evaluation as doctrine.** Every cross-date WER comparison must use the same sweep; previously attributed "nondeterminism" turned out to be 13 different checkpoints published in one day. The model is verified bit-deterministic across processes.
- **Rejection criteria written down.** `granite_qwen_frozen.yaml` is explicitly a *control experiment* — its header spells out in advance what each possible outcome would mean. That is a rare and excellent practice.
- The trade: extreme readability and honesty over breadth. One projector architecture, one data mix, no length-grouped batching (removed as config-unreachable, flagged as the next throughput lever). The comments occasionally outweigh the code.

## Comparison notes

- [[Whistle — On-Device Speech Recognition]] attacks the same problem from the opposite end: a 16.9 MB model designed from scratch for CPU-only on-device inference, versus Tiny Audio's assembled-from-pretrained-parts model needing ~6 GB. Whistle wins on footprint and latency; Tiny Audio wins on WER ceiling and on being *trainable by you for $25* — its point is that the training loop, not the model, is the product.
- [[SAM Audio]] and Tiny Audio both wrap foundation audio models in task-specific plumbing (isolation vs transcription/diarization), but Tiny Audio exposes the whole training stack where SAM Audio is consume-only.
- [[When Smaller Models Win]] argues small fine-tuned self-hosted models beat frontier general models on narrow tasks; Tiny Audio is the strongest concrete exhibit in the wiki for that thesis — 80M trained parameters against billion-parameter commercial ASR, with `ta eval` making the comparison falsifiable.
- Unlike [[Local Qwen Is Not a Worse Opus]]'s "different tool, not worse tool" framing for *inference*, Tiny Audio applies the same humility to *training*: the course recipe deliberately produces a worse model so learners run the full loop cheaply.

---
*Sources: [[raw/tiny-audio]], [[summary/tiny-audio]]*
*Last updated: 2026-10-11*
