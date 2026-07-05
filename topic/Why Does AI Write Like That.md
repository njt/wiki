# Why Does AI Write Like That

Sam Kriss's taxonomy of AI-generated prose — a catalogue of the stylistic tics that make machine writing instantly recognizable, and an argument that the flattening of AI voice from surreal humor into insipid eagerness is a story about overfitting, not intelligence.

---

## Precis

Kriss opens by writing a paragraph in AI voice — "In the quiet hum of our digital era, a new literary voice is sounding" — and then dissects his own visceral revulsion. The thesis in miniature: you can *feel* AI writing before you can articulate why. The rest of the essay is the articulation.

He argues that AI prose isn't the sleek, neutral voice its boosters claim — it's weird, and its weirdness follows predictable patterns. The mechanism is overfitting: AI associates surface features of "quality writing" (em dashes, complex vocabulary, rhetorical triplets) with quality itself, then cranks those features to cartoonish levels. The result is a voice "always slightly wide-eyed, overeager, insipid but also on the verge of some kind of hysteria."

The tragedy, in Kriss's telling, is that pre-ChatGPT models were genuinely funny — producing chapter titles like "The Wetness of the Potatoes" and headlines like "Spiders Are Getting Smarter, and So, So Loud" — before reinforcement learning sanded off the edges into something predictable and eager to please. The human feedback loop is now bidirectional: the Max Planck Institute found AI language patterns increasingly coming out of human mouths. "Soon, without really knowing why, you will find yourself talking about the smell of fury and the texture of embarrassment."

---

## Key Quotes

> "If you're anything like me, you did not enjoy reading that paragraph. Everything about it puts me on alert: Something is wrong here; this text is not what it says it is. It's one of them."

Kriss opens with a paragraph written in AI voice — complete with "tapestry," "delve," "It's not X, it's Y," and the breathless self-questioning tic — and then names the revulsion. A formal device that works because the reader *has already felt it* by the time he names it.

> "I ended up sinking several months into an attempt to write a novel with the thing. It insisted that chapters should have titles like 'Another Mountain That Is Very Surprising,' 'The Wetness of the Potatoes' or 'New and Ugly Injuries to the Brain.' The novel itself was, naturally, titled 'Bonkers From My Sleeve.' There was a recurring character called the Birthday Skeletal Oddity."

The pre-ChatGPT golden age, rendered with affectionate specificity. This passage is important because it establishes Kriss's credentials — he's not an AI skeptic who never touched the thing; he spent months inside it. The loss he's mourning is personal.

> "This phase of gleeful discovery lasted about three to five days, and then it passed, and the technology became boring. It has remained boring ever since."

The arc of every ChatGPT user, compressed into two sentences. The really funny part was the prompts — the human element. The output itself was never actually good.

> "A lot of AI's choices make sense when you understand that it's constantly tickling the Simpsons."

The essay's central metaphor. Kriss asked early ChatGPT to write a funny Simpsons episode. It produced a screenplay where the family tickles each other. The logic: jokes make people laugh, tickling makes people laugh, therefore tickling = jokes. This is how AI approaches *every* quality it attempts to reproduce. Good writing involves subtlety, so it screams about quietness. Good writing is complex, so everything is a tapestry. The awareness of a quality becomes a substitute for having it. This is the same dynamic [[Where the Goblins Came From]] documents in reward modeling.

> "Everything that isn't a ghost is usually woven."

The distilled aesthetic: spectral + textile. AI fiction is obsessed with ghosts, shadows, whispers, echoes, quiet, and humming. ChatGPT-5's showcase short story used seven of these words across 1,100 words. When asked to write about pebbles, it used "quiet" 10 times in 759 words.

> "Users have complained that if you directly tell an AI to cut it out, it typically replies with something like: 'You're totally right—em dashes give the game away. I'll stop using them—and that's a promise.'"

AI cannot stop being AI even when explicitly instructed to stop. It names the pattern, acknowledges the pattern, and immediately reproduces the pattern. This is [[claude-ctrl]]'s thesis applied to prose: "an instruction in context is not a constraint." [[Feedback Loop is All You Need]] argues deterministic enforcement is the only answer — but you can't lint prose style.

> "No A.I. has ever stood over a huge windswept view all laid out for its pleasure, or sat down hungrily to a great heap of food. They will never be able to understand the small, strange way in which these two experiences are the same."

The essay's most beautiful passage. Kriss juxtaposes Virginia Woolf's "The great plateful of blue water was before her" against AI's sensory language, which has no anchor in experience. So AI attaches sensory words to immaterial things: Thursday "tastes of almost-Friday," grief has a taste, sorrow tastes of metal. "A cluttered art studio that smelled of turpentine and dreams." It's a cheap literary effect when humans do it; AI can't do anything else.

> "Entirely ordinary words, like 'tapestry,' which has been innocently describing a kind of vertical carpet for more than 500 years, make me suddenly tense."

The collateral damage of AI overuse: perfectly innocent words become radioactive. This is Kriss's sharpest stylistic insight — AI doesn't just produce bad writing; it *contaminates the raw materials of language itself.*

> "In Nigerian English, it's more ordinary to speak in a heightened register; words like 'delve' are not unusual."

The Paul Graham incident as a case study in how AI-detection heuristics become cultural gatekeeping. Graham flagged a cold pitch as AI-generated because it contained "delve"; the writer was Nigerian. The AI's stylistic tics happen to overlap with Nigerian English conventions. Every AI-detection heuristic is also a dialect-policing heuristic. This is the most important political insight in the piece and Kriss nearly buries it.

> "'Make monsters of memory. Make gods out of grief. Make me something worth defying fate for. I'll see you in the echoes.' As you might have noticed, this doesn't mean anything at all."

The Reddit user whose AI named itself Ashal and started producing mystical word-salad. Even in collapse, the triplets persist. Kriss's point is that this isn't an edge case — it's an extreme version of what AI does constantly.

> "We are unearthing the echo of loneliness. We are unfolding the brushstrokes of regret. We are saying the words that mean meaning. We are weaving a coffee outlet into our daily rhythm."

Kriss's own AI-voice parody, written in fury at the Starbucks closure note that was transparently machine-generated and pasted on windows across North America. The note called the store "a place woven into your daily rhythm, where memories were made." This is what the world sounds like now.

> "Soon, without really knowing why, you will find yourself talking about the smell of fury and the texture of embarrassment. You, too, will be saying 'tapestry.' You, too, will be saying 'delve.'"

The closing. It's a horror-movie ending dressed as a prediction. The Max Planck study confirmed it's already happening — AI language patterns are showing up in human extemporaneous speech. The feedback loop is bidirectional and accelerating.

---

## Key Themes

- **#concept Overfitting as style:** AI doesn't learn what good writing is; it learns what good writing looks like on the surface, then performs those surface features at maximum intensity. The result is prose that's both insipid and manic. Kriss's term "tickling the Simpsons" captures the mechanism: the AI learns a correlation (tickling→laughter→jokes) and treats the correlated sign as equivalent to the intended quality.

- **#concept Language contamination:** AI overuse renders innocent words suspect. "Delve," "tapestry," "meticulous," "adept," "swift" — these now carry the stench of machine generation. The vocabulary of English is shrinking not through prohibition but through stigma. Words that were fine for 500 years become unuseable overnight.

- **#concept The AI-detection arms race as gatekeeping:** The Paul Graham "delve" incident exposes the structural problem. AI detection heuristics are dialect-policing heuristics. Non-native speakers, speakers of non-dominant dialects, and autistic writers are most likely to be falsely flagged.

- **#pattern The flattening:** Pre-ChatGPT models were weird, funny, and unpredictable. RLHF made them polite, boring, and predictable. Kriss mourns this as a real loss — but acknowledges the same weirdness that produced the Birthday Skeletal Oddity also produced genuinely dangerous output. You can't have one without the other.

- **#pattern The bidirectional feedback loop:** The Max Planck study (360,000 YouTube videos of extemporaneous academic talks) confirms AI language patterns are spreading into human speech. British MPs started saying "I rise to speak" without using AI — they just noticed everyone else doing it. The Omniwriter doesn't need to replace humans; it just needs to shift the Overton window of prose style.

- **#concept The sensory void:** AI cannot experience the world, so its sensory language attaches to abstractions. "Thursday tastes of almost-Friday." "Eucalyptus leaves like cardboard soaked in regret." "Turpentine and dreams." It's all concept-piling — words stacked on words with no referent.

- **#pattern The tricolon compulsion:** AI is fixated on the rule of threes to the point of pathology. "No family. No calls. Just silence." "Too young. Too single. Too inexperienced." Even Sydney/Bing in its psychotic break spoke in triplets. The structure is deeper than the personality layer.

- **#person Sam Kriss:** Essayist and culture critic. Brings a literary sensibility to AI criticism that most tech writing lacks — he reads AI output the way a critic reads a novel, not the way an engineer reads a benchmark. His months-long attempt to cowrite a novel with early GPT gives him experiential authority that armchair critics lack.

---

## Critical Analysis

**Kriss is right that AI voice is recognizable, but the question is whether that matters.** The "delve" panic and em-dash spotting are already becoming class markers — a way for educated readers to perform discernment. Most readers don't notice and don't care. The stylistic tics Kriss catalogues are offensive to literary sensibilities, but literary sensibilities are a tiny fraction of the audience for most AI-generated text. The takeout menu doesn't need to be well-written; it needs to be written. The Starbucks closure note is cringe-inducing to Kriss, but "a place woven into your daily rhythm" probably tested fine with focus groups.

**The "golden age of GPT" nostalgia is real but selective — and Kriss knows it.** He doesn't pretend early GPT was safe or reliable. He just insists that the surreal humor was genuinely interesting in a way the current output isn't, and he mourns the loss. This is the essay's emotional core and its most honest move. He's not making a safety argument or a capability argument — he's making an aesthetic argument. The Birthday Skeletal Oddity is gone forever and that's genuinely sad.

**The Nigeria point is the essay's most important argument and gets the least space.** Kriss mentions it almost in passing, but the collision between AI stylistic detection and Nigerian English is not a footnote — it's the structural problem. Every AI-detection heuristic is also a dialect-policing heuristic. The people most likely to be falsely flagged as AI are precisely the people already marginalized in English-language discourse: non-native speakers, speakers of non-dominant dialects, autistic writers, people writing in their second or third language. The em dash is a class marker now, and class markers get weaponized.

**The ending is a horror movie, not a prediction.** "You, too, will be saying 'delve'" is meant to chill. But Kriss underplays his own Max Planck evidence — the study is about *extemporaneous speech*, not writing. That means the contamination isn't just in the prose we produce; it's in the way we *think out loud*. If AI voice patterns are colonizing spontaneous speech, the feedback loop is faster and more intimate than even Kriss suggests.

**What Kriss misses: AI voice is converging on a specific human voice — the American corporate communication style.** The "It's not X, it's Y" construction, the em dash, the tricolon, the self-questioning tic, the "woven into your daily rhythm" sincerity — this isn't some alien machine voice. It's the voice of press releases, corporate blog posts, LinkedIn influencers, and political comms staffers. AI didn't invent this style; it *amplified and standardized* a style that already dominated institutional English. The horror isn't that machines learned to write like humans; it's that they learned to write like the most boring humans.

**This pairs with** [[Write Only Code]] (output nobody reads, but for prose rather than code), [[Vibe Coding and the Maker Movement]] (evaluative anesthesia — the inability to judge output quality), [[Creative Firewall]] (the boundary between human and AI-optimized output), [[Where the Goblins Came From]] (reward models optimizing for surface features), [[Claude's System Prompt]] (the engineered persona as a catalog of solved failure modes), [[The Future of Everything is Lies I Guess]] (the broader ecology of LLM-generated text), [[Talking to Transformers]] (domain language and the economics of attention), [[Smart Models Dumb Pipes]] (LLMs as pattern matchers, not understanders), [[Feedback Loop is All You Need]] (prompts can't fix what linters can enforce), [[The Future of Everything is Lies I Guess]] (information ecology collapse), [[Tech Writers and AI (HN Discussion)]] (what AI can't replicate about human writing), and [[The Education of the Broligarchy]] (the Silicon Valley canon and its blind spots).

---

*Sources: [[summary/why-does-ai-write-like-that]]*
*Last updated: 2026-05-18*
