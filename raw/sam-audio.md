---
url: https://ai.meta.com/research/samaudio/
title: SAM Audio — Segment Anything in Audio
author: Meta AI Research (Bowen Shi, Andros Tjandra, John Hoffman, et al.)
date_fetched: 2026-05-14
date_published: 2025-12
---

## Source Content

SAM Audio is a foundation model from Meta that isolates any sound from audio or audio-visual sources using text, visual, or temporal prompts. It's part of Meta's "Segment Anything" family (SAM 3 for images/video, SAM 3D for spatial understanding, SAM Audio for sound).

### Prompt Modalities

1. **Text prompts** — describe the target audio in natural language (e.g., "thunder", "man speaking")
2. **Visual prompts** — click on the part of a video where the target sound occurs
3. **Span prompts** — select a timespan containing the target audio (first model to introduce this)
4. **Multi-modal prompts** — combine text, visual, and timespan cues

### Domain Coverage

- **General Sounds** — everyday sounds like traffic or barking dogs
- **Music** — isolates instruments and vocals with high accuracy
- **Speech** — extracts speech from background noise, enables speaker isolation

### Architecture

Powered by a flow-matching Diffusion Transformer operating in a DAC-VAE latent space. It's a generative separation model that extracts both target and residual stems from an audio mixture. Multiple model sizes: small, base, large, plus -tv variants optimized for visual prompting.

### Companion Model: PE-AV

Perception Encoder Audio Video — a new open-source model bringing audio capabilities to Meta's Perception Encoder family. Detailed in a separate paper on large-scale multimodal correspondence learning.

### Evaluation

Releases an OSS evaluation set for prompted audio separation and a judge model correlated with human subjective evaluation. The judge assesses separation quality across precision, recall, and faithfulness. Re-ranking uses CLAP (text-audio similarity), Judge (separation quality), and ImageBind (visual-audio similarity).

### Accessibility Partnerships

- **2gether-International** (Diego Mariscal): AI as game-changer for disabled community
- **Starkey** (Achin Bhowmik): AI for hearing technology innovation

### Links

- Paper: https://arxiv.org/abs/2512.18099
- GitHub: https://github.com/facebookresearch/sam-audio
- Demo: https://aidemos.meta.com/segment-anything/editor/segment-audio
- Dataset: https://huggingface.co/datasets/facebook/sam-audio-bench
- HuggingFace collection: https://huggingface.co/facebook/sam-audio

### Citation

Bowen Shi, Andros Tjandra, John Hoffman, et al. "SAM Audio: Segment Anything in Audio." arXiv:2512.18099, 2025.
