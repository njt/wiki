# Quebec (Yarn's Forced Aligner)

Yarn (a video editor where you or agents edit by editing the transcript) needed word timings accurate to ~20 ms and found no forced aligner good enough, so they built Quebec: a fenced MFA decoder plus a 19 MB boundary net plus LLM-generated training labels, reaching 77% of word edges within 20 ms of a human label — close to the ~79% at which two humans agree, and far above MFA's 54%. The write-up is a model of measured, incremental ML engineering, and a quiet argument that agents editing media by transcript is a real product shape worth building infrastructure for.

---

## What it argues

Small, well-chosen constraints beat bigger models. The three stages each attack a distinct failure:

1. **The fence.** MFA's beam search throws away paths it can't get back, so when audio and transcript disagree for a second the right path is already gone. Fix: trust the transcriber's *sloppy but rarely wrong* timings as a fence, and only search within 200 ms of them — the search becomes small enough to solve exactly, with no pruning. Unaligned words drop from 1-in-12 to 1-in-500.
2. **The boundary model.** The acoustic model names the sound in each 10 ms slice; where phonemes overlap, either label scores about the same, so the boundary can fall anywhere in the blend. A tiny conv net (Whisper tiny, 2 layers) looks at 200 ms each side of one boundary and places it in 5 ms steps. Within-20ms goes 60% → 77%, median error 15 ms → 8 ms.
3. **LLM-generated labels.** Hand-labelled boundary data is paywalled; the one open set (Map Task) wasn't enough. Early attempts failed — spectrogram-in, boundary-out was worse than the decoder's own guess; coding agents drawing their own plots burned 100K tokens per 15 boundaries and were no better. What worked was *one* Gemini call with the decoder's guess, two zoomed plots, and a page of phonetic rules. Gemini labels within 20 ms of humans 69% of the time (vs 51% for the guesses it started from), and trained the net as well as human labels (73.9% vs 74.0%) at under $200 for ~17K boundaries.

## Key quotes

> It puts 77% of word edges within 20 ms of a human's; the next best gets 54%.

The number that justifies the whole post. "Within 20 ms of a human" and "within 20 ms of *two humans agreeing*" (79%) is the honest ceiling framing — they measure against human agreement, not against a fantasy of zero error.

> We first tried having an LLM referee between MFA and the transcriber. The best models picked the right one only a little better than chance.

A rare published negative result. The lesson generalises: LLMs are good at local perception tasks with the right representation (spectrogram plots), bad at adjudicating between competing systematic pipelines.

> For a word ending in a vowel, nasal or liquid, the word ends where the regular strong glottal pulses stop and the 500-4000 Hz energy collapses [...] the weak decaying low-frequency murmur [...] belongs to the pause.

The rule page that made Gemini labelling work: domain knowledge written as *checkable phonetic rules*, handed to the labeller alongside plots. The prompt is a tiny spec, not a vibe.

> Point your agent at this page to get started.

The API docs ship with an `llms.txt` twin and a one-liner telling you to aim your agent at them. Yarn's whole premise — agents editing video via transcript — means *agents are the aligner's primary customer*, and the docs are written accordingly.

## Key themes

- #pattern **Constrain, then solve exactly** — fence the search with an error model (transcriber rarely off by >200 ms) instead of pruning heuristically; each stage shrinks the next stage's problem until a tiny net suffices.
- #concept **Sloppy-but-unbiased beats precise-but-lost** — Scribe's timings are noisy yet almost never off by a lot; MFA is precise but loses its place. Composition beats either alone.
- #tool **LLMs as data labellers** — multimodal labelling with the decoder's guess as a prior, zoomed plots, and phonetic rules; validated three ways (vs humans, inter-model agreement, ablation against human labels) before training.
- #concept **Honest benchmarking** — hand-labelled ground truth, an unseen corpus (Buckeye) for transfer, human-agreement as the dashed ceiling line, and stated caveats (English only, built on MFA/Whisper components).

## Analysis

This is what good ML engineering writing looks like: a metric (boundaries within 20 ms, bad cuts), a ceiling (human agreement), an ablation table where each step shows its delta, a negative result (the LLM referee), and an unseen-corpus transfer test. The architectural moral — *make the problem small before making the model big* — is old, but the specific trick of fencing with an error profile of a different model is genuinely elegant, and the fence is what makes the cheap 19 MB boundary net viable at all: "by this point the problem is small."

The Gemini labelling result is the part with the widest implications. Labels that help training *as much as human labels* for $200 changes the economics of small-data perception problems — but note the conditions: a narrow, rule-describable task, a strong prior (the decoder's guess), and three validation checks before trusting the labels. It is the "human when cheaper" tier of [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] inverted: here the *model* is cheaper than the human and provably as good, but only because they measured. Without the validation checks, "as good as human labels" is exactly the kind of claim this wiki's verification pages exist to distrust.

There is also a product thesis hiding in the post. Yarn's editor cuts audio at gap midpoints based on word timings; that only works because they could rebuild the layer beneath their product when nothing on the market was good enough. "We figured somebody must have fixed this already, but found nothing better" is the recurring founder experience in the agent-tooling era: the stack is still full of 20%-off-the-ideal components, and the fix is often three unglamorous ideas and a measurement harness, not a foundation model.

## Related pages

- [[Whistle — On-Device Speech Recognition]] — the on-device counterpart: Whisper-style transcription with word timestamps from decoder attention, sharing the same problem (word-level times) at the opposite point of the local/cloud and size spectrum; Quebec's 19 MB boundary net shows how small you can go when the problem is fenced.
- [[SAM Audio]] — Meta's prompted audio separation, the big-model frontier of the same audio domain; Quebec is the counterpoint that a composed pipeline of small components can beat end-to-end systems on a narrow, well-measured task.
- [[Klangio Transcription Studio]] — another audio-understanding product built around transcription accuracy as the core quality bar, with a freemium demo gating on exactly the hardest slice of audio.
- [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] — Quebec complicates the tier rule: Gemini labelling beats the human tier on cost at equal quality, but only after three independent validations; the tiers are priced by verification cost, as that essay implies but doesn't demonstrate.

---
*Sources: [[raw/quebec-yarn-so]], [[summary/quebec-yarn-so]]*
*Last updated: 2026-11-20*
