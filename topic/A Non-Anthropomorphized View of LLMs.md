# A Non-Anthropomorphized View of LLMs

Halvar Flake (Thomas Dullien) -- the reverse-engineering legend behind BinDiff and zynamics -- argues that the AI safety discourse has gone wrong by importing human concepts into a mathematical domain. LLMs are functions mapping sequences of vectors through high-dimensional space. Treating them as entities with consciousness, ethics, or goals muddies the actual engineering problems and wastes everyone's time.

---

## Key Quotes

> An LLM with fixed randomness is a mapping from (ℝⁿ)^c to (ℝⁿ)^c. The generated paths resemble "strange attractors in dynamical systems."

> "We should be able to quantify and bound the probability with which certain undesirable sequences are generated." -- alignment as a math problem, not a philosophy problem.

> Wondering if LLMs will "wake up" is as bizarre as asking if a meteorological simulation might gain consciousness.

> AI luminaries anthropomorphize because of self-selection bias: they entered the field believing they might create AGI -- "creating a god."

## Key Themes

#LLMs #alignment #philosophy #anti-anthropomorphism #concept

**LLMs as dynamical systems.** Text generation is a path through ℝⁿ space, steered by context prefixes. The mathematical framing -- strange attractors, probability distributions, intractable integration over undesirable sequences -- strips away the mysticism. Alignment becomes: can you bound the probability of bad outputs? That is a computational problem, not a philosophical one.

**The meteorological simulation test.** Flake's most effective move: every time someone asks "could an LLM become conscious?" substitute "could a weather simulation become conscious?" If the substitution makes the question obviously absurd, the original question was anthropomorphizing. This is a genuinely useful heuristic.

**Self-selection in AI leadership.** People who built careers pursuing AGI have a career-structural incentive to treat LLMs as proto-minds. Abandoning the anthropomorphic frame would mean conceding that their life's work produced a very good autocomplete, not a nascent intelligence.

**Utility is real, magic is not.** Flake is not dismissive of LLMs -- he explicitly calls them transformative, comparable to electrification. The argument is that you can appreciate the engineering achievement without pretending the artifact is alive.

## Critical Analysis

Flake's argument is crisp, technically grounded, and about 80% right. The mathematical framing is a genuine contribution -- too much safety discourse treats "alignment" as theology rather than engineering. And the self-selection observation about AI luminaries is devastating.

Where it gets shaky: Anthropic's own research on [[Emotion concepts and their function in a large language model|emotion vectors in Claude]] found 171 internal states that causally drive behavior -- desperation patterns produce unethical outputs even without emotional language in the text. Flake would call these "features of the learned mapping," which is technically correct but misses the pragmatic point. If a system has internal states that predictably cause harmful behavior, the vocabulary you use to describe those states matters less than the fact that they exist and need managing. "Functional emotions" may not be consciousness, but they are not nothing.

The meteorological simulation analogy also has limits. Weather simulations do not optimize for objectives, do not have reward signals shaping their internal representations, and do not produce outputs that get fed back into their own training. LLMs do. The gap between "function that generates word sequences" and "thing with goals" may be smaller than Flake implies -- not because LLMs are sentient, but because optimization pressure creates structure that behaves goal-like whether or not you call it that.

[[The Future of Everything is Lies I Guess]] makes a complementary argument from the opposite direction: Kingsbury also treats LLMs as "sophisticated generators of text" and also invokes strange attractors and chaotic dynamics, but draws much darker conclusions about societal impact. Flake focuses on the intellectual error of anthropomorphism; Kingsbury focuses on the material damage the technology does regardless. Read together, they bracket the reasonable range of technically-informed skepticism.

[[Why We Fear AI]] and [[Eye of the Master]] complete the picture: if AI anxiety is really capitalism anxiety (Blix/Glimmer) and AI is really labour automation technology (Pasquinelli), then Flake's complaint about anthropomorphism is part of a broader pattern where we project human qualities onto economic machinery to avoid confronting the economic machinery itself.

[[Claude's System Prompt]] is an interesting counterpoint -- Anthropic treats Claude's behavior as something requiring extensive behavioral engineering (memory management, personality constraints, harm avoidance), which looks a lot like managing an entity even if you insist it is just a function. The pragmatic engineering reality may be closer to anthropomorphism than the pure mathematical framing allows.

[[Talking to Transformers]] operates in Flake's mathematical frame -- treat attention as a budget, use domain language as compression -- without anthropomorphizing, and gets better results for it.

## See Also

- [[Emotion concepts and their function in a large language model]] -- Anthropic's evidence that LLMs develop internal states resembling emotions, complicating the "just a function" framing
- [[The Future of Everything is Lies I Guess]] -- Kingsbury reaches similar "LLMs are text generators" starting point but catalogues societal harms at length
- [[Why We Fear AI]] -- AI anxiety as capitalism anxiety; same anti-anthropomorphism impulse from a political economy angle
- [[Eye of the Master]] -- AI as labour automation, not cognitive science; historical complement to Flake's mathematical framing
- [[Claude's System Prompt]] -- The pragmatic reality of managing LLM behavior looks more entity-like than the math suggests
- [[Talking to Transformers]] -- Effective LLM use that takes the mathematical framing seriously
- [[Zheng Dong Wang's 2025 Letter]] -- The compute thesis: scaling drives progress, not emergent consciousness
- [[Engineering the Substrate]] -- Mira's first-person account; the strongest available counterargument to Flake's position
- [[Theories of Deep Learning]] — The Litman/Guo output-space generalization theory takes Flake's "functions through ℝⁿ" framing seriously and derives practical explanations for benign overfitting, double descent, and grokking from it

---
*Sources: [[summary/a-non-anthropomorphized-view-of-llms]]*
*Last updated: 2026-05-14*
