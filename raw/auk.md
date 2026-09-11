---
url: https://huggingface.co/tencent/AuK
date_fetched: 2026-09-11
---

## News

- **[2026/09/09]**🎉- **AuK**is now open-source. Code and model weights are publicly available. Try it on the 🤗 Demo Space or the 🤖 ModelScope Space!

## Contents

- News
- Introduction
- Performance
- Model Architecture
- Supported Tasks
- Download the weights
- Citation
- License

## Introduction

**AuK** is a 1.5B foundation model for speech generation and editing. Trained on millions of hours of diverse audio data, AuK supports zero-shot and
instruction-based TTS, content and acoustic editing, paralinguistic editing,
speech enhancement, and source separation through a unified natural-language
instruction interface. AuK has two variants:

| Model | Description | Weight | 
|---|---|---|
| AuK | Base model for high-quality generation | 🤗 Hugging Face · 🤖 ModelScope | 
| AuK-Flash | Distilled model for fast 4-step inference | 🤗 Hugging Face · 🤖 ModelScope | 

This repository contains the official weights for **the AuK base model**.

## Performance

## Model Architecture

## Supported Tasks

AuK exposes every task through the same natural-language instruction interface. The table below groups the supported tasks by category, with a short description and a link to its section in the Cookbook, which provides instruction templates plus CLI and Python examples.

| Category | Task | Description | Cookbook | 
|---|---|---|---|
| Speech Generation | Zero-shot TTS | Speak the target text in the voice of the reference audio. | Zero-shot TTS | 
| Instruct TTS | Generate speech from a voice description alone — no reference audio. | Instruct TTS | |
| Content Editing | Speech Content Editing | Rewrite what is said— replace, insert, or remove text. | Speech Content Editing | 
| Lyric Editing | Rewrite lyrics in a singing recording while preserving the melody and voice. | Lyric Editing | |
| Acoustic Editing | Pitch Editing | Raise or lower the pitch by semitones. | Pitch Editing | 
| Speed Editing | Adjust the speaking rate; output length scales with the speed factor. | Speed Editing | |
| Volume Editing | Raise or lower the volume by decibels. | Volume Editing | |
| Paralinguistic Editing | Emotion | Change the emotion while preserving content and voice. | Emotion | 
| Timbre | Change the timbre to a description while keeping the content unchanged. | Timbre | |
| De-accent | Remove a regional accent while preserving the speaker's voice and content. | De-accent | |
| Nonverbal Editing | Remove or add nonverbal sounds such as breaths, laughs, or coughs. | Nonverbal Editing | |
| Whisper Conversion | Convert between normal speech and whisper while preserving speaker and content. | Whisper Conversion | |
| Enhancement & Separation | Speech Enhancement | Denoise, dereverberate, or restore natural, clear speech. | Speech Enhancement | 
| Speech Separation | Keep one speaker by talking order and remove the others. | Speech Separation | |
| Music Separation | Extract the singing voice from a mix, or keep all human voices. | Music Separation | |
| Target Speaker Extraction | Keep the target speaker identified by what they say. | Target Speaker Extraction | 

## Download the weights

**🤗 HuggingFace**

```
pip install -U "huggingface_hub[cli]"
# AuK-Base
hf download tencent/AuK --local-dir ./ckpts/AuK
# AuK-Flash (4-step distilled) 
hf download tencent/AuK-Flash --local-dir ./ckpts/AuK-Flash
# MLLM Encoder
hf download Qwen/Qwen2.5-Omni-3B  --local-dir ./ckpts/Qwen2.5-Omni-3B
```
**🤖 ModelScope**

```
pip install -U modelscope
# AuK-Base
modelscope download --model Tencent-Hunyuan/AuK --local_dir ./ckpts/AuK
# AuK-Flash (4-step distilled)
modelscope download --model Tencent-Hunyuan/AuK-Flash  --local_dir ./ckpts/AuK-Flash
# MLLM Encoder
modelscope download --model Qwen/Qwen2.5-Omni-3B --local_dir ./ckpts/Qwen2.5-Omni-3B
```
The expected directory structure is:

```
ckpts/
├── AuK/
├── AuK-Flash/          # optional
└── Qwen2.5-Omni-3B/
```
The model checkpoint contains the diffusion transformer and layer-fusion weights. The MLLM encoder and VAE are loaded from separate files at runtime, so missing `text_encoder.*` keys during checkpoint loading are expected.

For installation, inference, Gradio, ComfyUI, and fine-tuning, see the GitHub README and Cookbook.

## Inference with SGLang-Omni

Thanks to the SGLang Omni team for completing Day 0 support for AuK. You can launch SGLang Omni with the following command:

```
python -m sglang_omni.cli serve --model-path tencent/AuK
```
For detailed usage and configuration options, refer to the SGLang Omni AuK Cookbook and the optimization roadmap of AuK.

## Citation

Research using AuK can cite the following entry:

```
@misc{ma2026auktechnicalreportopensource,
  title={AuK Technical Report: An Open-Source Foundational Model for Speech Generation and Editing},
  author={Ziyang Ma and Zhikang Niu and Wenming Tu and Tianrui Wang and Ruiqi Yan and Junxi Liu and Yanru Huo and Nickk Huang and Yang Liu and Qicong Xie and Zeyu Xie and Hui Wang and Haitao Li and Zixuan Jiang and Yalin Li and Jie Fang and Yifan Duan and Zeyue Tian and Guangzheng Li and Haina Zhu and Shuyi Wang and Jinwen Wang and Mingyu Cui and Tian Tan and Auden and Sen Liang and Steve Yves and Shan Yang and Liefeng Bo and Zilong Zheng and Kai Yu and Eng-Siong Chng and Xie Chen},
  year={2026},
  eprint={2609.08936},
  archivePrefix={arXiv},
  primaryClass={cs.SD},
  url={https://arxiv.org/abs/2609.08936},
}
```
## License

AuK is released under the **MIT License**. See `LICENSE` for the full terms.

- Downloads last month
- 179
