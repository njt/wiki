# Pocket TTS

A 100-million parameter text-to-speech model from Kyutai that includes voice cloning and runs on your laptop's CPU. No GPU required.

---

## Key Themes

#ai #text-to-speech #voice-cloning #edge-computing

The remarkable thing is the size: 100M parameters. For context, most capable TTS models are 1B+ parameters and require GPU acceleration. Pocket TTS claims comparable quality at a fraction of the size, running on CPU alone. If the quality holds up, this changes the economics of voice synthesis entirely -- embedded devices, offline applications, and privacy-sensitive deployments all become feasible.

Voice cloning at this scale is particularly interesting. If you can clone a voice with a 100M parameter model running on a laptop CPU, the barrier to personalized TTS drops to essentially zero. Custom voices for accessibility tools, personalized audiobook narration, language learning with familiar voices -- all without cloud APIs or GPU rental.

Kyutai (funded by Iliad Group, CMA CGM Group, Schmidt Sciences) is positioning itself as the lab for efficient speech models. Their research portfolio spans text-to-speech, speech-to-text, and speech-to-speech, all with an emphasis on running locally.

## Critical Analysis

The inverse of [[Handy]] -- where Handy turns speech into text on-device, Pocket TTS turns text into speech on-device. Together they represent the full local voice pipeline: speak, transcribe, process, synthesize speech back. No cloud round-trip needed.

The 100M parameter claim needs benchmarking against larger models. "Runs on CPU" and "sounds good" are in tension -- the question is where on the quality-efficiency curve Pocket TTS lands. For accessibility and utility applications, "good enough" is genuinely good enough. For professional voice work, probably not.

The voice cloning capability also raises obvious misuse concerns. A model this small and portable is easy to deploy maliciously. Kyutai presumably has thoughts about responsible release, but the genie is hard to contain at 100M parameters.

---
*Sources: [[raw/pocket-tts]]*
*Last updated: 2026-05-14*
