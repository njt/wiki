# How to Follow a Drummer

Sashyo's field report on building DrumMate, an Android app that inverts the forty-year tyranny of the click track: the drummer leads, the machine follows. The hard-won engineering lessons — phase-locked loops with beat-aware front ends, the division-is-free/multiplication-is-prediction asymmetry, and why coasting isn't optional — are platform-agnostic and sharper than most academic treatments of real-time tempo tracking.

---

## Key Quotes

> "A hit should be treated as evidence, not as a command. Following a drummer, not obeying one."

This one sentence contains the entire architecture. Every naive approach — kick-as-clock, tap-tempo averaging, phase-reset-on-hit — fails because it treats drum hits as orders to be executed. The working system treats them as noisy observations of a hidden state (tempo + phase) and updates a belief. It's a Bayesian frame disguised as a musician's aphorism.

> "Division is free, multiplication is prediction."

The asymmetry that "took embarrassingly long to see" is the article's sharpest technical insight. Clock division is causal and bulletproof because it never needs to know the tempo — it just counts. Clock multiplication — arpeggiators, ratchets, synced delays — secretly carries a tempo model whether the designer admits it or not. The cheap ones assume a BPM floor and silently fail below it. The honest ones solve the estimation problem. This distinction alone is worth the read for anyone building real-time music systems.

> "Small deviations are feel, sustained drift is tempo."

The sensitivity tightrope in one sentence. Chase every hit and you quantize away the pocket — the drummer's deliberate micro-timing becomes wobble. Ignore everything and you miss the chorus push. The rule discriminates on *persistence*, not magnitude. This is the same insight behind every good Kalman filter: process noise vs. measurement noise, separated by whether the deviation *persists*.

> "The fix was coasting: when confidence wobbles but hits are still arriving, the clock keeps running on its last good estimate and waits."

The most demoralizing bug — band locks in, then vanishes the moment the drummer does anything interesting — and its fix. A human bandmate doesn't stop playing during a fill; they hold their groove and listen harder. The algorithm should do the same. Sashyo calls coasting "not optional" for any version of this system, and I believe him — a follower that gives up during the interesting parts is worse than no follower at all.

> "This whole problem is really about teaching the machines the musicianly thing, instead of making the humans play machinely — which is what click tracks and quantized backing have been doing to us for forty years."

The thesis, stated plainly at the end. Forty years of recording technology have trained musicians to follow machines. DrumMate inverts that. The article's title could have been "How to Make a Machine Follow a Drummer" — but it's not, because the hard part isn't the machine, it's understanding what drummers actually do.

## Key Themes

- **#concept Evidence vs. command** — Drum hits are noisy observations of hidden state, not triggers. The estimation layer fits onsets against a grid hypothesis (period + phase + downbeat) and nudges proportionally. Onsets that disagree — fills, syncopation, ornaments — are mostly ignored. This is a phase-locked loop with a beat-aware rejection front end, and it's the right architecture for any live tempo tracker.

- **#pattern Coasting** — When confidence wobbles but signal is still present, maintain state on the last good estimate rather than stopping or chasing noise. Sashyo presents this as the single most important lesson: his worst version failed because it treated confidence dips as silence. The fix — "keep running and wait" — is what human bandmates do instinctively and what most algorithms get wrong.

- **#pattern Prediction, not reaction** — The system schedules notes against a *forecast* of the next beats, not in response to each hit. This keeps audio latency out of the drummer's timing loop entirely. It's also what a human rhythm section does: nobody waits to hear beat one before playing beat one. The architectural implication is that a follower needs a tempo model whether you want one or not — the only question is how honest you are about it.

- **#tool DrumMate** — Android app: plug in an e-drum kit over USB MIDI, a generated band (bass, keys, lead) follows your tempo, dynamics, and feel. Free beta at drummate.app. Built by Sashyo & Petty Vendetta.

- **#person James Holden** — Referenced for his human timing work and Max patches that go further than DrumMate: bidirectional entrainment where machines and humans pull on each other as coupled oscillators. DrumMate is deliberately one-way (drummer leads, band follows), but Holden's direction — mutual influence rather than master-slave — is the more interesting long-term research program.

## Critical Analysis

**The PLL-with-rejection architecture is correct and under-described.** Sashyo sketches the estimation layer in a few paragraphs: hits arrive as timestamps, a grid hypothesis (period + phase) is maintained, agreeing onsets nudge proportionally, disagreeing onsets are ignored. This is a phase-locked loop with outlier rejection — standard signal processing, but almost nobody applies it to tempo tracking because the music-tech world defaults to tap-tempo averaging. The article would benefit from a block diagram or pseudocode, but the intuition is clear enough to implement from.

**The "human tapper" admission is the most honest thing here.** After all the clever engineering, Sashyo concedes that a human tapping along is still the best tempo tracker. Brains predict, algorithms react. The only reason to build this in software is that a drummer can't be their own tapper and not every jam has a spare human. This kind of epistemic honesty — "here's what I built, here's why it's still worse than a person" — is vanishingly rare in technical writeups. It's also correct: the human auditory system's ability to predict beat one through a fill is something no real-time algorithm touches. Whether transformer-based models trained on drumming corpora could close this gap is an open question the article doesn't explore.

**The DJ software comparison is crucial context most readers will miss.** Traktor and Rekordbox get the whole track up front, run offline analysis, fix mistakes in a second pass. A live tracker has none of that — no lookahead, no undo, commits in real time. The engineering gap between offline beat detection (solved) and online tempo following (hard) is enormous, and Sashyo is right to name it explicitly. Most people who've used DJ software don't understand why live tracking is a different problem.

**The James Holden contrast deserves more space.** Sashyo mentions Holden's bidirectional entrainment work — coupled oscillators where machines and humans pull on each other — and correctly notes his system is deliberately one-way. But the one-paragraph treatment undersells the significance. Holden's approach, grounded in Hennig et al.'s research on correlated timing deviations with memory, suggests that the hard problem isn't following a drummer — it's building a machine that grooves *with* a drummer. Mutual entrainment is a harder problem than master-slave following, and the fact that Holden has working Max patches suggests it's tractable. Sashyo's one-way design is correct for DrumMate's premise (the drummer is the boss), but the interesting frontier is bidirectional.

**The platform choice is both pragmatic and limiting.** Building this as an Android app with USB MIDI input targets e-drummers specifically — a small but real market. But as Sashyo notes, the core problem is platform-agnostic: a Eurorack module, a Max patch, a plugin would all benefit from the same architecture. The Android constraint means acoustic drummers (who'd need audio-to-MIDI conversion) are excluded, and the generated-band approach means this is a practice/performance tool rather than a production one. A VST plugin version that follows recorded drums and drives sequenced instruments would reach a much larger audience.

**The "forty years" claim is rhetorically effective but historically incomplete.** Click tracks have indeed dominated recording since the 1980s, but drummers pushing and pulling against the grid — and engineers accommodating it — is older than Pro Tools. The history of tempo mapping in DAWs, of "humanize" functions in drum machines, of the whole "make machines sound human" project, is the mirror image of Sashyo's "make machines follow humans" project. Both are reactions to the same problem. DrumMate inverts the approach (follow rather than simulate) but doesn't escape the frame.

---

## Connections

- [[Moises — AI Music Separation and Creation]] — Both are AI/ML-powered music tools that process audio in real time (or near-real time). Moises separates stems from a mix; DrumMate extracts tempo from hits. The shared challenge: making machine listening fast enough to be musically useful.
- [[Klangio Transcription Studio]] — Transcription is the offline version of tempo tracking: given a full recording, find the notes and rhythms. DrumMate does it live and only needs tempo + downbeat; Klangio does it offline and needs pitch + timing for every note. Same problem family, different latency budget.
- [[ytx How to Write Interesting Chord Progressions]] — The wiki's other music-theory page. Harmony vs. rhythm — the two axes of music that get independent tooling and independent theory. The radial model (seven strands radiating from a key centre) is conceptually similar to DrumMate's estimation layer: multiple independent signals feeding a central hypothesis.
- [[Waveloop]] — Music visualization that reveals harmonic structure. Adjacent space: tools that make musical structure visible/actionable rather than just audible.

---

*Sources: [[raw/how-to-follow-a-drummer]]*
*Last updated: 2026-07-11*
