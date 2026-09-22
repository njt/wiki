# Piano Autocomplete — On-Device Music Copilot

A Show HN post describing a 125M-parameter transformer that autocompletes piano performances in real time (~108 notes/sec on an iPhone 15, entirely on-device via Core ML). The pitch is "GitHub Copilot or Tabnine for music": prompt with a few MIDI notes and the model continues the phrase. The author grounds the idea in 18th-century schema theory — Gjerdingen's *Gebrauchs-Formulas* — and a Rachmaninoff dinner-party recording of composers playing a pattern-generation game, arguing that next-note prediction is simply next-token prediction applied to music.

---

## Key Quotes

> "The idea is basically GitHub Copilot or Tabnine, except instead of prompting it with code, you prompt it by playing a few notes on a MIDI piano."

The framing matters because it demystifies the model. This isn't "AI composes music"; it's autocomplete — the same completion interface programmers now treat as ordinary, transplanted to a different sequence domain. The author borrows an existing metaphor rather than inventing a grander one, which is honest about what the model is and is not doing.

> "I trained a 125M-parameter transformer to autocomplete piano performances in real time (~108 notes/sec on an iPhone 15)."

The two numbers are the whole technical claim. 125M parameters is small by frontier standards — comparable to [[TimesFM]]'s 200M and an order of magnitude below even a compact TTS model like [[Pocket TTS]]. And it runs on a phone, in real time, with no cloud. For an interactive instrument that latency constraint is the point: you cannot autocomplete a live performance through a network round-trip.

> "Happy to answer questions about the model, training, Core ML, or the many things that didn't work."

The Show HN honesty clause. "The many things that didn't work" is the part worth trusting — it signals a real training run with real dead ends, not a demo engineered to a happy path.

> "I'd highly recommend reading Robert Gjerdingen's article Gebrauchs-Formulas."

This is the most interesting move in the post. Gjerdingen's galant-schema / partimento work describes how 18th-century composers internalized reusable musical "formulas" and combined them fluently — a pattern library, not a blank slate. The author is claiming that what the transformer learns is precisely that library of formulas. The model isn't inventing; it's re-acquiring a centuries-old practice of pattern completion.

## Key Themes

- **#pattern Autocomplete as a general interface** — Copilot, Tabnine, and now a piano. Completion is becoming the default shape of generative-AI interaction across domains: prompt with a prefix, accept or reject a continuation. Music is the cleanest test of whether that interface generalizes beyond code.
- **#tool On-device inference** — Core ML on an iPhone 15 at ~108 notes/sec. The latency budget of live performance forces the model small and local, the same constraint [[Inflect-Micro-v2]] exploits for TTS and [[Bonsai 27B]] pushes toward for LLMs.
- **#concept Next-note prediction as next-token prediction** — treating MIDI as a token stream. This is the exact move [[TimesFM]] makes for time series (patch tokenization → next-patch prediction) and [[TabFM (Tabular Foundation Model)]] makes for tables.
- **#concept Musical schemata (Gjerdingen)** — the historical claim that composition was already pattern completion. The AI framing is a rediscovery, not a breakthrough.
- **#person Sergei Rachmaninoff** — invoked (with three other Russian composers) as evidence that expert musicians played this game for fun in the 1890s, audiating from sheet music without a piano.

## Critical Analysis

**The honest framing is the strongest part of the post.** Calling it "Copilot for music" and pointing at Gjerdingen does two things at once: it sets expectations (suggestions, not compositions) and it locates the technology in a real intellectual tradition. Most music-AI launches reach for "creative partner" or "the future of songwriting"; this one reaches for "autocomplete" and an 18th-century pattern theory. That restraint is rarer than the model.

**But music is a domain where "next" is underdetermined.** Code completion has a compiler and tests as a ground-truth backstop; the model's job is bounded. Music has vastly more valid continuations, no compiler, and taste as the only arbiter. A 125M-parameter model predicting the next note is really proposing one plausible continuation among many — fine for a suggestion tool, fatal if anyone mistakes it for an oracle. The post mostly avoids that mistake.

**The on-device choice is the product, not an optimization.** ~108 notes/sec on a phone means the play-hear-suggest loop is tight enough to be part of playing, the way Tabnine's suggestions are part of typing. Cloud-based music generation ([[Moises — AI Music Separation and Creation]], [[Suno Training Data Breach]]) can afford minutes of latency because its artifacts are finished stems and songs. An autocomplete that kept you waiting would be useless. The author implicitly understands this: the thing only works because it is small and local.

**The missing piece is the corpus.** "A 125M-parameter transformer" says nothing about what it was trained on — whose piano, whose MIDI, which styles, which licensing regime. The music-AI training-data fight ([[Suno Training Data Breach]]) is the shadow hanging over every on-device music model too, even a hobbyist's. The post doesn't address it, and that omission is the one place where the honesty runs out.

## Connections

- [[TimesFM]] — the same architecture move in another non-text sequence domain: decoder-only transformer, sequence patching, next-token prediction. Piano MIDI is to this model what time series is to TimesFM.
- [[Inflect-Micro-v2]] — a complete single-modality model at a fraction of frontier scale, running locally. 9.4M params for speech, 125M for piano: the compact-model pattern is becoming a genre.
- [[Moises — AI Music Separation and Creation]] — the cloud, 65M-user pole of music AI. This is its mirror image: one developer, one instrument, no server.
- [[Klangio Transcription Studio]] — the reverse direction around MIDI. Transcription reads what was played (audio → notes); autocomplete predicts what comes next (notes → continuation). Both make MIDI the machine interface to music.

---

*Sources: [[raw/piano-autocomplete-on-device-music-copilot-show-hn]], [[summary/piano-autocomplete-on-device-music-copilot-show-hn]]*
*Last updated: 2026-09-04*
