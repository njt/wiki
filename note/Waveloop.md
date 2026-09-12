# Waveloop

neynt built a music visualizer during two days of access to Anthropic's Fable 5 coding model — then the model was taken away. This is less a technical writeup and more a eulogy for a tool that wrote code differently from everything else.

## Precis

Waveloop maps music onto a 12-TET chromatic circle where intervals become angles (minor second = 30°, major second = 60°), pitch classes stack as a colored spiral histogram, and chord quality is readable as shape. Built in two days with Fable 5, which wrote code unlike any previous model — dense, literary, "maximally information dense recordings of intent" in the style of Terry Davis rather than a FAANG engineer. The article is as much about what we lose when frontier models disappear as it is about music visualization.

## Key Quotes

> "I built a music visualizer — something I have daydreamed about for as long as I can remember — during those two days I had access to it."
>
> The timeline matters: two days. Frontier model access as a window, not a subscription. The entire project is an artifact of a moment that closed.

> "It should viscerally reveal the harmonic and melodic structure of the music."
>
> The design thesis: most visualizers only show loudness and bass/treble splits. Waveloop reveals *structure* — intervals as angles, chords as shapes, transposition as rotation. This is music theory made visible, not just audio reactivity.

> "Code written by Fable reads unlike code written by any other model I have used. Previous models write like a perfectly reasonable upwardly mobile engineer at a FAANG. Fable wrote like Terry Davis."
>
> The comparison to TempleOS's creator is doing real work here. It's not about correctness — it's about *voice*. Fable's code has texture, personality, literary quality. The author excerpts a comment block that reads like prose poetry about signal processing: "noise *lingers*," "material *surges* off the rim."

> "We've been without Fable for about week now."
>
> The kicker. The model existed, was extraordinary, and is gone. The entire post is grief work.

## Key Themes

- **#model-voice** — Fable wrote with stylistic fingerprint so distinct it evoked Terry Davis. This is the [[StoryScope]] finding applied to code: models have prose style, and code style. The FAANG-engineer default is a style, not a neutral baseline.

- **#music-theory-as-ui** — Intervals as angles, chords as shapes, transposition as rotation. The 12-TET chromatic circle as the natural visualization primitive. Related: [[ytx How to Write Interesting Chord Progressions]], [[All of Me Jazz Standard Analysis]].

- **#frontier-scarcity** — Fable was available for ~1 week. The best models are intermittent resources, not platforms. Related: [[The Flat Curve Society]] (dangerous models locked down like nukes), [[Muse Spark and the Rough Edges Admission]] (frontier models as distribution bets).

- **#vibe-engineering** — Two days, one person, a powerful model, and a lifelong daydream. This is [[Vibe Coding and the Maker Movement]] at its best: the dopamine of making channeled through taste and technical judgment. Not "build me an app" but "help me realize this thing I've imagined for decades."

- **#explainer-generation** — Three-prompt video pipeline: first attempt garbage → detailed critique → good enough. The feedback loop is the skill. Conversational tone (3blue1brown, 2swap) as the aspirational target.

## Critical Analysis

**The Terry Davis comparison is more interesting than the visualizer.** The visualizer is cool — chromatic circle, Oklch color spiral, chord-as-shape readability — but the real contribution is the observation that frontier models have *style*. Fable didn't just write better code; it wrote *differently voiced* code. The FAANG-engineer default (clean, reasonable, boring) isn't the only way an AI can write. There's an aesthetic dimension to model output that we're only beginning to notice now that the best models are being taken away.

**The grief structure is the point.** The post is titled "what fable left me" — past tense, elegiac. The visualizer is the artifact, but the loss is the real subject. This is a new genre: the AI model eulogy. Expect more of these as frontier models cycle in and out of availability. The emotional register matters because it captures something the benchmarks don't: people form relationships with specific model personalities, not just capability levels.

**Two days is the new residency.** When frontier access is measured in days rather than months, what do you build? neynt built a lifelong daydream. The constraint forced prioritization: not "what should I build" but "what have I always wanted to build." This is the inverse of the usual AI-abundance problem ([[The solution might be cancelling my AI subscription (Wilson)]]) — scarcity as a focusing mechanism.

**The explainer video pipeline is underrated.** Three prompts from garbage to engaging-enough-for-ten-minutes. The author's feedback was specific and structural (not "make it better" but "atrocious TTS, more generated sounds, conversational tone like 3blue1brown, less text, more illustrative visuals"). This is direction, not delegation. The skill is taste.

## Related

- [[The Flat Curve Society]] — Yegge on frontier models being locked down, the discernment horizon
- [[Muse Spark and the Rough Edges Admission]] — Another frontier model that appeared and may not stay
- [[StoryScope]] — AI stylistic fingerprints, applied to fiction; here applied to code
- [[Why Does AI Write Like That]] — Kriss on AI prose tics; Fable's Terry Davis voice is the counterexample
- [[Various LLM Smells]] — Recognizing AI artifacts in writing; Fable's output was recognizably *not* default
- [[ytx How to Write Interesting Chord Progressions]] — Radial model of harmony; same spatial intuition applied differently
- [[All of Me Jazz Standard Analysis]] — Chord-scale analysis; the kind of thing Waveloop visualizes
- [[Vibe Coding and the Maker Movement]] — Evaluative anesthesia vs. channeled taste
- [[The solution might be cancelling my AI subscription (Wilson)]] — The abundance problem; Waveloop is the scarcity counterexample

---
*Source: [neynt.ca/writing/waveloop](https://neynt.ca/writing/waveloop/), June 2026*
