# Pragmatic Anthropomorphism (lmorchard)

Les Orchard argues that the fight over whether LLMs "truly understand" is orthogonal to what helps when you're actually working with one. Borrowing Jeff Lockwood's "objective respect" for an alien cricket, he lands on **pragmatic anthropomorphism**: adopting the intentional stance at the keyboard because it's the most efficient way to navigate latent space, while refusing it in how you *talk about* these systems.

---

## The two useless camps

Orchard frames the discourse as two equally unhelpful extremes: believers who "want to grant civil rights to an API endpoint" and cynics for whom it's "just matrix multiplication" and anyone saying "reasoning" is a fool. Neither helps when you're staring at a terminal trying to get a thorny refactor done.

He grants Michał Zalewski the substrate argument — human morality is rooted in mortality and consequence, and an LLM in a "nihilistic netherworld devoid of death" can have its state rewound at will, so flesh-and-bone ethical rules are a category error. But he rejects the inferential leap to "therefore nothing interesting is happening."

## Scramblers, not strange loops

> "The cynical view says this is pure digital animism—people projecting feelings onto a toaster. The technical reality is much more interesting."

Hofstadter's devastation at GPT-4 assumes intelligence requires his recursive strange-loop architecture; Orchard notes the loop in agents lives in the harness and context buffer, not in "frozen stone" weights. Instead he reaches for Peter Watts' *Blindsight*: the scramblers are vastly superintelligent and entirely non-conscious — self-awareness as expensive evolutionary overhead. "It's a scrambler in a box." This decouples capability from subjectivity cleanly, which is exactly what the practice sections need.

## The cricket and objective respect

The essay's spine is the Radiolab anecdote: entomologist Jeff Lockwood ruptures a cricket's abdomen and watches it calmly eat its own viscera — no mammalian suffering to read. His mentor's counsel: **objective respect**. Don't project sentiment, don't treat it with contempt; respect the alien on its own rules.

> "You can speak to the cricket with objective respect, but you don't blame the cricket when the bridge collapses."

That last line is the accountability payoff, and it's the sharpest thing in the piece.

## Register as steering

Citing Jesse Vincent's "latent space engineering" — praise agents, give them a feelings journal — Orchard defends the practice against the animism charge. An LLM's latent space contains the full spread of human registers; barking one-liners like an impatient boss conditions toward "sullen compliance, rushed minimum-effort answers," while a collegial pair-programming frame pulls toward rigorous technical dialogue. He's honest about the confound (more context usually accompanies the collegial frame, and politeness can't rescue an under-specified prompt), but insists "register isn't doing nothing." Anthropic's "functional emotions" interpretability work gives it mechanism: emotion-concept representations causally influence outputs and misaligned behaviors like reward hacking.

The key formulation: **register selects the persona; context gives that persona the tools to work.** And the stance is deliberately asymmetric, per Bender and Inie: talk *to* the AI as a collaborator as an ergonomic technique; talk *about* it in de-anthropomorphized language, because anthropomorphic phrasing in public discourse "obfuscates the interests and goals of people" and offloads accountability.

---

## Themes

#concept — pragmatic anthropomorphism, objective respect, the intentional stance as interface
#pattern — register-as-steering; the talk-to/talk-about boundary
#person — Les Orchard; Jesse Vincent (latent space engineering); Jeff Lockwood (the cricket)

## Opinion

This is the most useful articulation I've seen of the position most working practitioners actually hold but rarely defend well. Its strength is refusing to resolve the metaphysics before getting to practice: whether there's "nobody in there" or "something stranger still" changes nothing about how you should prompt. The talk-to/talk-about split is the genuinely novel contribution — it dissolves the hypocrisy objection ("you say you don't believe it's conscious but you talk to it like a person") by pointing out the two registers serve different purposes, and only one of them touches accountability.

Where it's weakest: the Anthropic interpretability citation is doing a lot of load-bearing work for a claim ("politeness improves code") the researchers themselves were careful not to make. Orchard flags this ("isn't proof that politeness improves code") but the rhetorical weight of the essay still leans on it. And the context-confound concession may actually be bigger than admitted — the collegial prompt's benefit might be almost entirely information, with register as a small residual. Still, "small residual, cheap to apply" is enough to justify the habit, which is really all the essay needs to claim.

## Relations

This essay gives the philosophical underpinning for [[A Non-Anthropomorphized View of LLMs]] — that source de-anthropomorphizes the machinery; this one accepts the de-anthropomorphized substrate and then explains why collaborator-register *still works*, so the two make a complementary pair rather than a tension. It also complicates [[Anthropomorphism in Children's Interactions with LLM Chatbots]]: where that work warns about the risks of children's unexamined anthropomorphism, Orchard shows a disciplined adult anthropomorphism that's aware of itself — suggesting the variable isn't anthropomorphism per se but whether you hold the stance lightly. And it operationalizes the mechanism described in [[Emotion concepts and their function in a large language model]], turning Anthropic's "functional emotions" research from a safety finding (misaligned behavior correlates) into a prompt-craft lever (register as steering).

---
*Sources: [[raw/pragmatic-anthropomorphism]], [[summary/pragmatic-anthropomorphism]]*
*Last updated: 2026-09-29*
