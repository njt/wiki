# Tone LLM: LLM-Driven Guitar Tone Generation

Vishwanath Subramanian's `tonellm` converts text descriptions of guitar tone into `.pdpreset` files for the Polychrome DSP amp simulator. The reusable insight isn't the guitar domain — it's the **contract/adapter pattern**: the LLM fills a small Pydantic JSON schema, and deterministic Python translates that JSON to proprietary plugin XML. The model never touches the plugin format. This is the cleanest worked example I've seen of a pattern that generalizes to *any* LLM→proprietary-config problem.

---

## Key Quotes

> "The LLM is a *proposal generator*; you are the approver."

This is the design philosophy in one line. The LLM doesn't produce the final artifact — it produces an editable intermediate representation. The `.tone.json` sidecar is the human-editable layer of the contract. Subramanian designed for the case where the LLM gets it *mostly* right and the human tweaks the rest. Most LLM tools design for the fantasy case where the LLM gets it perfectly right and the human never looks. This alone makes the architecture worth studying.

> "I was deeply humbled... pitting my limited pre-existing tones on amp sims versus the settings an LLM can spit out which sound devastatingly better."

The author's humility about his own domain expertise is notable. He's a guitarist and programmer who built a tool, fed it his own text descriptions, and found the LLM's output materially better than what he'd dialed in by hand. This isn't AI hype — it's a practitioner discovering that the model's pattern-matching across decades of forum posts, manuals, and interviews produces results that beat individual trial-and-error.

> "The system prompt is the curriculum."

Not fine-tuning. Not RAG. A markdown file loaded as the system prompt containing era cheat tables, guitar compensation rules, and explicit myth-busting ("Heavy ≠ scooped mids"). This is prompt-as-knowledge-base, and it works because the domain is bounded enough that a single system prompt can hold the relevant expertise. For domains too large for a single prompt, you'd need retrieval — but for guitar tone, a few pages of markdown suffices. The constraint is the feature.

> "A single `client.chat()` call beats agent tool loops for this task."

The author tried ReAct chains and agent loops and found them "a bit muddy and overcomplicated." The final design is one chat completion returning one JSON object. This is a useful counterexample to the instinct to agent-ify everything. When the task is "fill in a bounded form from natural language + optional audio features," a single structured-output call is faster, cheaper, and more debuggable than a multi-step agent. The engineering discipline is recognizing when your problem fits this shape.

---

## Key Themes

- **#pattern Contract/adapter architecture for LLM output** — The four-layer recipe: (1) define a small schema, (2) put domain rules in system prompt, (3) put variable evidence in user prompt, (4) generate with schema constraint + validate + deterministic codegen. This generalizes to any proprietary config format: DAW presets, CI pipelines, k8s manifests, game engine configs. The LLM fills the contract; the adapter translates. [[OpenAI Structured Outputs]] provides the schema enforcement at the API level; this project shows how to compose it with domain-specific translation.

- **#tool Prompt-as-curriculum** — The `tone_system.md` file encodes domain expertise (era cheat tables, guitar compensations, myths to avoid) as a system prompt rather than fine-tuning or RAG. This works when the domain is small enough to fit in context. The interesting question is where the boundary is: how much domain knowledge can you pack into a system prompt before the model loses focus? For guitar tone, the answer is "enough."

- **#pattern Human-editable sidecar as the contract surface** — The `.tone.json` file is the interface between LLM output and deterministic translation. Users can edit it and re-run translation without another API call. This separation means: the LLM is disposable (swap vendors, change models), the contract is durable (validated by Pydantic), and the translator is deterministic (zero LLM involvement). This is [[Smart Models Dumb Pipes]] applied to creative tools.

- **#concept Audio feature extraction as LLM prompt augmentation** — Librosa extracts interpretable features (spectral centroid, crest factor, band energy) and converts them to natural-language paragraphs appended to the user prompt. This is a pattern worth stealing: use classical DSP to measure the thing, describe the measurements in prose, let the LLM reason about them. It's cheaper and more interpretable than training a multimodal model on audio.

- **#tool Four-layer defense for structured output** — Schema constraint at generation time → text extraction → Pydantic validation → deterministic translator. Each layer catches what the previous one can't. This is defense-in-depth for LLM output, and it's more robust than any single layer alone. The [[Guardrails and Feedback Loops]] thesis in microcosm: linters beat prompts, and multiple linters beat one.

---

## Critical Analysis

**The architecture is more valuable than the application.** The guitar tone generator is cool, but the contract/adapter pattern is the enduring contribution. Every proprietary config format — every DAW plugin, every game engine preset, every enterprise software export — could use this approach. Define a small JSON schema, write a deterministic translator, and let the LLM fill the JSON. The LLM never needs to know the target format exists. This is the inverse of most "LLM writes code/config" approaches, which try to have the LLM generate the target format directly and then spend forever debugging syntax errors.

**The "prompt-as-curriculum" approach has unexamined limits.** Subramanian's system prompt is 2-3 pages of markdown. This works for guitar tone because the domain is bounded: there are only so many amp archetypes, cab types, and era conventions. But the approach doesn't scale — you can't fit the equivalent of a textbook into a system prompt. The implicit design decision is "constrain the domain to what fits in one prompt" rather than "build a retrieval system." This is the right call for a personal tool, but it limits the tool's scope. You can't add "and also model bass amps and drum kits and synth patches" without either bloating the prompt past the model's attention horizon or switching to RAG.

**The single-call design is a feature, not a limitation.** The author's rejection of agent loops is correct for this task, but the reasoning deserves interrogation. Agent loops fail when the task is *form-filling from a description* — there's nothing to iterate on, no tool to call, no intermediate result to inspect. The LLM either produces the right descriptor or it doesn't. Adding tool calls would introduce failure modes without adding capability. The general lesson: agent loops add value when the task requires *interaction* (calling tools, inspecting results, adjusting approach). They subtract value when the task is *completion* (fill this bounded form from this description). Knowing which is which is the engineering skill.

**The honest limits section is refreshing and should be standard.** Subramanian lists five specific limitations (full mixes lie about guitar character, cab mappings calibrated by ear, presets are starting points, not a replacement for playing, tone is in the fingers). This is what good engineering documentation looks like — not "our AI does everything" but "here's exactly what it does, here's exactly what it doesn't, here's how to work around the gaps." Every AI tool should have a section like this.

**The audio analysis layer is underbaked but promising.** The librosa feature extraction → natural language paragraph → prompt augmentation pipeline is clever but limited. Six features (centroid, rolloff, flatness, crest, three band energies, tempo) is a coarse sketch of an audio signal. The real opportunity is feeding the raw features directly to the LLM alongside the prose description, or using embeddings from a pre-trained audio model. The current approach is "describe the measurements in English and hope the LLM interprets them correctly" — which works but loses information in the translation. A more direct approach would give the model structured feature vectors alongside the text query and let it learn the mapping.

**Comparison to other AI×music tools:** Unlike [[Moises — AI Music Separation and Creation]] (stem separation as the wedge into generation) or [[SAM Audio]] (prompted audio separation as research), Tone LLM doesn't process audio to produce audio. It processes text (and optional audio features) to produce *configuration*. It's an AI tool for a *human* making music, not an AI that makes music. This distinction matters: the former empowers the musician; the latter replaces them. Subramanian's framing ("not Mutt Lange in a box") shows he understands this. The tool saves knob-tweaking time so you can play more guitar. That's the right value proposition for AI in creative tools: accelerate the craft, don't automate the artist.

---

## Connections

- [[OpenAI Structured Outputs]] — The protocol-level schema enforcement that makes the contract layer reliable. Tone LLM uses Ollama's equivalent, but the pattern is the same: constrain the output space before generation
- [[Smart Models Dumb Pipes]] — The contract/adapter pattern is a pure expression of this philosophy: the model makes the creative decisions (what tone?), the pipes enforce the contract and translate to the target format
- [[Guardrails and Feedback Loops]] — The four-layer defense (schema → extraction → validation → translation) is guardrail thinking applied to LLM output. Each layer is deterministic; the LLM only participates in layer 0 (generation)
- [[Moises — AI Music Separation and Creation]] — The other major AI×music tool. Moises processes audio; Tone LLM produces configuration. Complementary: Moises could isolate a guitar stem, Tone LLM could generate the tone to re-amp it
- [[SAM Audio]] — Audio separation research. Tone LLM's librosa feature extraction is a much simpler approach to the same problem (understanding audio content), optimized for LLM consumption rather than separation quality
- [[Swamp Club]] — Zod-typed models and DAG execution as the agent-native pattern. Similar philosophy: typed schemas as the contract, deterministic execution around model calls
- [[All of Me Jazz Standard Analysis]] — Musician-as-craftsman content. Tone LLM automates the gear-tweaking so the musician can focus on the kind of craft this page documents
- [[Klangio Transcription Studio]] — Transcription as AI music tool. Tone LLM is the inverse direction: instead of audio→notation, it's description→configuration
- [[ytx How to Write Interesting Chord Progressions]] — Music theory content in the wiki. The "prompt-as-curriculum" pattern in Tone LLM (system prompt = domain knowledge) is conceptually similar: encode the rules, let the model apply them

---

*Source: [[summary/lm-guitar-tone-generator-polychrome]], https://vishsubramanian.me/lm-guitar-tone-generator-polychrome/*
*Fetched: 2026-06-15*
