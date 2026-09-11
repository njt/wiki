# AuK — Speech Foundation Model

AuK is Tencent Hunyuan's 1.5B-parameter open-source (MIT) foundation model for speech generation and editing. Where the field has historically built one model per task — a TTS model, a denoiser, a stem separator, a voice converter — AuK collapses all of them into a single flow-matching transformer that takes a natural-language instruction and an optional reference clip. It's the "Segment Anything" bet applied to speech: one model, one prompt interface, fourteen-plus tasks. The most interesting engineering is not the scale but the recipe — frozen LLM encoder with learned layer fusion, a BigVGAN+VITS audio VAE, prefix-conditioned editing, and a distilled 4-step variant that embodies the distillability thesis from [[Continuous Diffusion Language Models]].

---

## Architecture

Three frozen-or-small components compose at inference (`src/auk/infer/infer_auk.py`):

1. **Frozen Qwen2.5-Omni-3B encoder.** Text and reference audio go through a frozen multimodal LLM (`Qwen2_5OmniThinkerForConditionalGeneration`, vision tower deleted). Crucially, AuK does *not* fine-tune this 3B model — it learns an **ELMo-style layer fusion**: a softmax over per-layer learnable logits averages all hidden states, multiplied by a scalar (`CFMEdit.layer_weights` / `layer_scale` in `src/auk/model/cfm_edit.py`). Two parameters buy most of the benefit of the LLM's representations at a fraction of the fine-tuning cost.

2. **BigVGANFlowVAE.** Audio is compressed by a 480× VAE (`src/auk/model/vae/bigvgan_flow_vae.py`) into 64-dim latents, normalized by a global mean/std. It's a hybrid: a BigVGAN-style decoder (anti-aliased multi-periodicity "AMP" blocks with SnakeBeta activations, HiFi-GAN lineage) fronted by a VITS residual-coupling normalizing flow. The VITS flow makes the latent approximately Gaussian — which is exactly what flow matching wants.

3. **Flux2Edit transformer** (`src/auk/model/flux2_edit.py`). A Flux2Audio variant that consumes *pre-encoded LLM text embeddings* rather than raw text. Two phases: 8 double-stream **MMDiT blocks** (joint text-audio attention, per-stream qk-norm and RoPE), then 24 single-stream **DiT blocks** over the concatenated text+audio sequence. Conditioning is AdaGN-style: a timestep embedding modulates every norm via SiLU projections (the Flux "adaLN" pattern in `src/auk/model/modules.py`).

Conditional flow matching (`CFMEdit`) integrates the velocity field with an ODE solver (`torchdiffeq`, Euler/midpoint), classifier-free guidance, and an optional "sway sampling" cosine perturbation of the time grid. Only the transformer and the layer-fusion weights train; LLM and VAE are frozen throughout.

## Key techniques

- **Editing as prefix continuation.** The reference audio is VAE-encoded and *prepended in the sequence dimension* to the noised target latent; the ODE state is target-only (`CFMEdit.sample`, `flux2_edit._embed_audio`). "Replace this phrase" is implemented as *conditioned generation with the source as a prefix*, not a diff. This is what lets content editing, timbre editing, and separation all share one forward pass — a senior engineer coming from image editing would recognize it as the inpainting/prefix trick, but unified for every audio task.

- **One message interface, no task routing.** Every task is a ChatML `messages` list; the instruction carries the task. Text-only TTS appends a `|<no_prompt_audio>|` marker to the user turn rather than branching code (`infer_auk.generate`). No task heads, no classifier gating the model — the same design discipline Meta used for [[SAM Audio]], extended from separation to generation.

- **Distillation as a locked recipe.** AuK-Flash is a DMD-distilled student pinned to `nfe=4`, `cfg=0`, and a hardcoded `t_grid = [0.0, 0.076, 0.293, 0.617, 1.0]` — the code *ignores* caller-supplied sampling knobs and, per the comment, re-adding CFG "blows up the amplitude (clips hard)" because the student bakes in its own guidance. This is the continuous-diffusion distillability advantage from [[Continuous Diffusion Language Models]] made concrete: few-step flow maps work where discrete token correlation would collapse.

- **Duration as a first-class input.** `gen_seconds` maps directly to latent length (`ceil(seconds * sr / downsample)`); without it, the CLI estimates duration from the byte-length ratio of `ref_text` to `gen_text` (`get_gen_duration`). Duration prediction is folded into the *format*, not a separate model.

- **The Prompt Enhancer is a structured-output agent** (`src/auk/infer/pe.py`, 2000+ lines + a 1500-line `pe.config.yaml` task taxonomy). An LLM classifies a free-form request into one of ~14 task types, normalizes the instruction against per-task templates, estimates duration, then deterministic code runs VAD trimming and loudness normalization and prints a ready-to-run `auk-infer` command. It's the [[Tone LLM]] / structured-output pattern: the LLM fills slots, deterministic code does everything mechanical.

- **Frame-length dynamic batching** (`src/auk/train/train.py`). Training samples are sorted by latent frame length and greedily packed to a `frames_threshold` — variable-length audio batches with minimal padding, plus EMA and a per-t validation-loss curve that fixes `x0` noise across the t-grid so the curve reflects time alone.

## Design decisions

- **Optimized for breadth, not a single-task ceiling.** One 1.5B model doing everything will lose to a dedicated denoiser or a specialized TTS on that task. The bet is that a shared instruction interface and shared representations beat fourteen bespoke models on maintenance, data efficiency, and composability. This is the same trade [[SAM Audio]] makes, and AuK extends it: it generates speech from text, which SAM Audio cannot.

- **Frozen LLM + frozen VAE, small trainable core.** Keeping the 3B encoder and the VAE frozen means the fine-tuning pipeline (`[train]` extra) only touches the transformer — cheap enough to ship to users. The cost is inference-time: every generation still pays for a full Qwen2.5-Omni forward pass, so AuK needs a GPU, unlike a [[Inflect-Micro-v2]] that runs TTS in 9.4M params on CPU.

- **Flexibility for the base, discipline for the student.** AuK exposes NFE/CFG/sway/t_grid knobs; AuK-Flash hard-locks them to the distilled recipe. That asymmetry is a product decision: power users tune, casual users can't footgun themselves into clipped audio.

- **Heavy reuse of known components.** Flux2 blocks, BigVGAN, VITS flows, x-transformers RoPE, Qwen's processor — nearly nothing here is novel architecture. The novelty is the *assembly*: the layer-fusion-over-LLM, the prefix-conditioned edit, and the distillation recipe. It's a foundation model built from off-the-shelf LEGO, which is why it could be released MIT with day-0 SGLang-Omni support.

## Comparison notes

- **[[SAM Audio]]** is the closest sibling: a flow-matching DiT audio foundation model with open weights and natural-language prompting. But SAM Audio only separates (target + residual); AuK generates, edits, enhances, *and* separates, adds zero-shot/instruct TTS, and uses a BigVGANFlowVAE rather than a DAC-VAE. Where SAM Audio's headline was its span-prompting and judge model, AuK's is the single-instruction-interface breadth.
- **[[Inflect-Micro-v2]]** is the opposite pole of the TTS spectrum: 9.4M params, one fixed voice, VITS-family, CPU-only. AuK's VAE is VITS-descended too, but the models answer different questions — Inflect asks "smallest possible TTS," AuK asks "one model for every speech task." Inflect wins on deployability; AuK wins on voice cloning and editing.
- **[[Continuous Diffusion Language Models]]** — AuK is the audio instantiation of Dieleman's distillability thesis. AuK-Flash's 4-step DMD distillation with CFG-off is exactly the "few-step sampling comes more naturally to the continuous setting" advantage, and sway sampling / `t_grid` are the concrete noise-schedule knobs that essay describes in the abstract.
- **[[Moises — AI Music Separation and Creation]]** productizes one slice of what AuK open-sources: music separation is a single task row in AuK's cookbook, available as MIT-licensed weights rather than a 65M-user cloud service.

#tool #project #speech #audio #foundation-model #tts #diffusion #multimodal #flow-matching

---
*Sources: [[raw/auk]], [[summary/auk]]*
*Last updated: 2026-09-11*
