# Turning Your AI Into a World-Class Designer (Chimala)

Anshu Chimala explains why AI-generated design looks like slop and gives a repeatable three-stage process (Discover, Define, Deliver) — random seed strings, ambitious taste-laden prompts, a design-critic subagent loop, image and video generation, and human-driven subtraction — that turns coding agents into distinctive designers.

---

## The diagnosis: predictability is the enemy

LLMs are next-token predictors trained on human preference ratings, so at every design decision they pick the token most likely to please everyone. Great design is the opposite: it bends rules, takes risks, aims for emotion. Chimala's framing:

> "It's like the ultimate case of design-by-committee."

> "The problem is that the model can't inherently act randomly. It can only predict the most likely token. If we want variety, we have to bring it from outside the model."

That second line is the load-bearing insight of the whole piece: variety must be *imported* (seed strings, human taste) rather than requested, because asking a model to "be random" just samples tokens that sound random.

## Key quotes and commentary

> "We didn't ask the model to do anything unique or varied, so it makes sense that it keeps falling back on the same patterns it knows well. But just asking for variety doesn't work."

The demonstration is unarguable: four instances of the same model given the same prompt produce the same purplish-gradient landing page; asking for randomness produces the *same* different page every time. Anyone who has generated a website with an agent has seen this.

> "Your work is only complete when the critic independently deems it 9/10 or higher. Do not put that criterion in the critic prompt; keep it objective in its scoring."

The critic loop is the strongest engineering content here: a fresh-context subagent sees only the screenshot, evaluates against a studio-quality bar, penalizes overdone AI patterns, and gives a 0–10 score; a cheap implementer loops until the bar is met. The critic is under 10% of output tokens — taste is cheap when it only makes executive decisions. The warning about stopping criteria (an unsatisfied critic makes the agent "helplessly burn tokens") is exactly the failure mode of unbounded judge loops.

> "AI loves to add more, but it rarely takes away… it's risky to strip down a design and delete code. The model needs a push from you."

The Deliver section is really about humans supplying the courage the model lacks. His calorie-tracker before/after is convincing: "clean, minimalist design" was already in the prompt, and the model still shipped pink glows and custom buttons worse than native iOS components.

## Themes

- #concept — creativity as escaping the predictable token; randomness must be imported from outside the model
- #pattern — fresh-context critic subagent with objective scoring and a hard stopping criterion
- #tool — image and video generation (OpenAI/Gemini keys, fal.ai aggregation, chroma keying, video matting) as design materials, not novelties
- #person — Anshu Chimala, Apple R&D design lead applying human-designer management lessons to agents

## Analysis

This is one of the most actionable pieces on agent *aesthetics* rather than agent *correctness*. Its credibility comes from Chimala's Apple pedigree — the Double Diamond adaptation is grounded in having actually managed designers, and the "look over your design and ask what needs to be there" step is something no model currently does unprompted.

Two caveats. First, the piece is prompt-recipe journalism: the before/after screenshots are hand-picked demos, and no failure rates are given for the critic loop, which certainly burns tokens on designs that never converge. Second, the API-key handling advice (paste keys into the prompt, or stash them in a gitignored `.env.agents` referenced from CLAUDE.md) is casual about a real security surface — even a tight-spend key can be exfiltrated by a prompt injection into anything the agent reads.

Still, the core techniques generalize beyond visual design: seed strings for variety, objective external critics with a quality bar, expensive-judge/cheap-worker cost split, and subtraction as a human-only contribution. It strengthens the case that the differentiator in agent work is now taste, not access to the model.

## Related pages

This refines [[What You Bring to AI Determines the Result]] with concrete mechanics for the "bring your own taste" claim. The critic subagent is a design-domain instance of the judge loops in [[The Lifecycle of LLM-as-a-Judge]], with the same Goodhart risks. [[Waveloop]] observed that frontier models have aesthetic style; Chimala shows how to *elicit* it reliably. [[Claude Chic]] covers the same "AI output is visibly AI" problem from a culture angle; Chimala supplies the fix. The cheap-worker/expensive-critic split echoes [[Thrifty (Tiered Delegation for Claude Code)]].

---
*Sources: [[raw/how-to-turn-your-ai-into-a-world]], [[summary/how-to-turn-your-ai-into-a-world]]*
*Last updated: 2026-10-10*
