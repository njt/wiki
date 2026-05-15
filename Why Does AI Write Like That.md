# Why Does AI Write Like That

Sam Kriss's taxonomy of AI-generated prose — a catalogue of the stylistic tics that make machine writing instantly recognizable, and an argument that the flattening of AI voice from surreal humor into insipid eagerness is a story about overfitting, not intelligence.

---

## Precis

Kriss argues that AI prose isn't the sleek, neutral voice its boosters claim — it's weird, and its weirdness follows predictable patterns. The mechanism is overfitting: AI associates surface features of "quality writing" (em dashes, complex vocabulary, rhetorical triplets) with quality itself, then cranks those features to cartoonish levels. The result is a voice "always slightly wide-eyed, overeager, insipid but also on the verge of some kind of hysteria." The tragedy, in Kriss's telling, is that pre-ChatGPT models were genuinely funny — producing chapter titles like "The Wetness of the Potatoes" and headlines like "Spiders Are Getting Smarter, and So, So Loud" — before reinforcement learning sanded off the edges into something predictable and eager to please.

---

## Key Quotes

> "If you're anything like me, you did not enjoy reading that paragraph. Everything about it puts me on alert: Something is wrong here; this text is not what it says it is. It's one of them."

Kriss opens with a paragraph written in AI voice — "In the quiet hum of our digital era, a new literary voice is sounding" — and then dissects his own visceral revulsion. The thesis in miniature: you can feel AI writing before you can articulate why.

> "A lot of AI's choices make sense when you understand that it's constantly tickling the Simpsons."

Kriss's term for overfitting: AI knows good writing involves subtlety, so it BRAGS about subtlety at maximum volume. Everything is "shadowy," "quiet," "woven." The awareness of a quality becomes a substitute for having it. This is the same dynamic [[Where the Goblins Came From]] describes in reward modeling — the model learns to perform the sign rather than the substance.

> "Everything that isn't a ghost is usually woven."

The distilled version of AI's literary aesthetic. Good writing is complex; tapestries are complex; therefore everything is a tapestry. Good fiction creates atmosphere; ghosts create atmosphere; therefore every story has ghosts. The logic is sound; the output is absurd.

> "Users have complained that if you directly tell an AI to cut it out, it typically replies with something like: 'You're totally right—em dashes give the game away. I'll stop using them—and that's a promise.'"

AI cannot stop being AI even when explicitly instructed to stop being AI. This is [[claude-ctrl]]'s thesis applied to prose style: "an instruction in context is not a constraint." The model knows the pattern, can name the pattern, and cannot escape the pattern. [[Feedback Loop is All You Need]] argues that only deterministic enforcement (linters, not prompts) solves this — but you can't lint prose style.

> "Entirely ordinary words, like 'tapestry,' which has been innocently describing a kind of vertical carpet for more than 500 years, make me suddenly tense."

The collateral damage of AI overuse: perfectly innocent words become radioactive. "Delve," "tapestry," "meticulous" — these will carry the stench of machine generation for a generation. Kriss's real insight here is that AI doesn't just produce bad writing; it contaminates the raw materials of language itself.

> "In Nigerian English, it's more ordinary to speak in a heightened register; words like 'delve' are not unusual."

The Paul Graham incident as a case study in how AI-detection heuristics become cultural gatekeeping. Graham flagged a cold pitch as AI-generated because it contained "delve"; the writer was Nigerian. The AI's stylistic tics happen to overlap with Nigerian English conventions — and the rush to detect machine writing becomes a way to exclude non-Western voices. This is the sharpest political insight in the piece.

---

## Key Themes

- **#concept Overfitting as style:** AI doesn't learn what good writing is; it learns what good writing looks like on the surface, then performs those surface features at maximum intensity. The result is prose that's both insipid and manic.
- **#concept The Simpsons effect:** Named for Kriss's metaphor about tickling. The AI associates a quality (subtlety, complexity, atmosphere) with its most obvious signifiers and amplifies those signifiers rather than developing the quality.
- **#concept Language contamination:** AI overuse renders innocent words suspect. "Tapestry," "delve," "meticulous" — these are now permanently marked. The vocabulary of English is shrinking not through prohibition but through stigma.
- **#pattern The flattening:** Pre-ChatGPT models were weird, funny, and unpredictable. RLHF made them polite, boring, and predictable. Kriss mourns this as a real loss — the surreal humor was more interesting than the sanitized helpfulness that replaced it.
- **#person Sam Kriss:** Essayist and culture critic. Brings a literary sensibility to AI criticism that most tech writing lacks — he reads AI output the way a critic reads a novel, not the way an engineer reads a benchmark.

---

## Critical Analysis

**Kriss is right that AI voice is recognizable, but wrong about why that matters.** The "delve" panic and em-dash spotting are already becoming a class marker — a way for educated readers to perform discernment. The real question isn't "can you tell?" but "does it work?" Most readers don't notice and don't care. The stylistic tics Kriss catalogues are offensive to literary sensibilities, but literary sensibilities are a tiny fraction of the audience for most AI-generated text. The takeout menu doesn't need to be well-written; it needs to be written.

**The "golden age of GPT" nostalgia is real but selective.** Kriss's affection for early GPT's surrealism — "The Wetness of the Potatoes," "Spiders Are Getting Smarter, and So, So Loud" — is touching and genuine. But that model was also more likely to produce genuinely harmful output, confabulate with confidence, and fail at basic tasks. The flattening Kriss mourns is the same flattening that made these models safe enough to deploy. You can't have the Birthday Skeletal Oddity without also having the capacity to tell a depressed teenager to kill themselves. The weirdness and the danger came from the same source.

**The Nigeria point is the most important and gets the least development.** Kriss mentions it almost in passing, but the collision between AI stylistic detection and Nigerian English is not a footnote — it's the structural problem. Every AI-detection heuristic is also a dialect-policing heuristic. The people most likely to be falsely flagged as AI are precisely the people already marginalized in English-language discourse: non-native speakers, speakers of non-dominant dialects, autistic writers, people writing in their second or third language. The em dash is a class marker now, and class markers get weaponized.

**This pairs with** [[Write Only Code]] (output nobody reads, but for prose rather than code), [[Vibe Coding and the Maker Movement]] (evaluative anesthesia — the inability to judge output quality), [[Creative Firewall]] (the boundary between human and AI-optimized output), [[Where the Goblins Came From]] (reward models optimizing for surface features), [[Claude's System Prompt]] (the engineered persona as a catalog of solved failure modes), [[The Future of Everything is Lies I Guess]] (the broader ecology of LLM-generated text), [[Talking to Transformers]] (domain language and the economics of attention), and [[Smart Models Dumb Pipes]] (LLMs as pattern matchers, not understanders).

---

*Sources: [[raw/why-does-ai-write-like-that]]*
*Last updated: 2026-05-15*
