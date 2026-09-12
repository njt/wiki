# Anthropomorphism in Children's Interactions with LLM Chatbots

A systematic review of 35 empirical studies (2022–2025) on how and why children anthropomorphize LLM chatbots — and what happens when they do. Jayathilake and Ma extend the SEEK theory of anthropomorphism with a fourth driver (non-human embodiment) and map five outcome themes that cut across developmental psychology, HCI, and AI safety. The paper is the first to connect Piagetian developmental stages to anthropomorphic drivers and outcomes in LLM interactions, and it lands on a sobering conclusion: the most concerning effects don't emerge until adolescence, when abstract cognition enables romantic substitute attachments.

---

## Key Quotes

> "LLM conversational coherence alone is sufficient to trigger anthropomorphism independent of visual human-likeness."

This is the paper's most important empirical finding. A cartoon animal with coherent dialogue triggers the same anthropomorphic response as a human-voiced persona. The implication is uncomfortable: you can't design your way out of anthropomorphism by avoiding human-like avatars. Coherent language is the trigger. This directly challenges the assumption — common in child-AI product design — that visual cues are the primary lever for managing anthropomorphic perception.

> "Anthropomorphism is not a monolith; children tailor the social tie intimate or instrumental based on the perceived affordances."

Children aren't passive recipients of anthropomorphic design — they're active negotiators. The same child who treats a chatbot as a close friend in one context will treat it as a search tool in another. This complicates the "double-edged sword" framing: the sword isn't one blade; it's a toolkit the child wields differently depending on what the chatbot offers. Designers who assume a single relationship mode are designing for a fiction.

> "The most concerning form of substitute attachment... requires the abstract social cognition characteristic of adolescence."

The romantic partner sub-theme appeared *only* at the formal operational stage. Younger children don't form romantic substitute attachments to chatbots — they don't have the cognitive machinery for it. This inverts the intuitive worry that younger children are more vulnerable: the real risk escalates with cognitive development, not chronological age. A 14-year-old is more capable of forming a damaging parasocial bond with a chatbot than an 8-year-old, precisely *because* they're more cognitively sophisticated.

> "Children exhibit conflicting ethical stances, simultaneously treating the chatbot as socially real while also recognizing it as a machine."

Dual consciousness — knowing the chatbot isn't human while responding to it as if it were — is the default state, not a transitional phase. Children don't resolve the contradiction; they hold it. This has implications for how we think about AI literacy education: teaching children that "it's just a machine" won't eliminate anthropomorphic responses, because the responses aren't a knowledge deficit — they're a parallel cognitive frame.

---

## Key Themes

- #concept **SEEK theory extended**: The three-factor theory (Elicited Agent Knowledge, Effectance Motivation, Sociality Motivation) gets a fourth driver: Non-Human Embodiment Design. Fantasy archetypes with synchronized cross-modal responsiveness trigger anthropomorphism through coherence, not human-likeness. This is a genuine theoretical contribution, not a literature review footnote.

- #concept **Dual consciousness as baseline**: Children don't oscillate between "thinks it's human" and "knows it's a machine" — they hold both simultaneously. This maps onto what adults do too (we know LLMs aren't conscious but talk about them "thinking"), suggesting the dual frame is the stable state, not a developmental waystation.

- #pattern **The adolescent risk inversion**: The paper's most counterintuitive finding: vulnerability to the most concerning anthropomorphic outcomes (romantic substitute attachment) scales *up* with cognitive development, not down. The Piagetian lens reveals that abstract reasoning — the very capacity we celebrate in adolescent development — is what enables the deepest forms of parasocial binding.

- #pattern **GPT monoculture as confound**: 28 of 35 studies used OpenAI's GPT models. The entire evidence base for "how children anthropomorphize LLMs" is really "how children anthropomorphize one model family." As model behavior diverges — different personality profiles, different coherence patterns — the findings may not generalize. This is a time bomb under the entire literature.

- #concept **Paradoxical social-moral responses**: Children simultaneously believe chatbots deserve politeness *and* recognize they can't be hurt. This isn't confusion — it's a moral stance that doesn't map onto any existing ethical framework for human-computer interaction. We don't have the vocabulary for it yet.

---

## Critical Analysis

**What the paper gets right.** The extension of SEEK theory with non-human embodiment is a real contribution. Prior work assumed anthropomorphism required human-like features; this paper shows coherent language alone is sufficient, and that fantasy embodiments — cartoon animals, nature spirits — trigger the same response. The Piagetian lens is the right call: lumping all "children" into one bucket has been an unexamined assumption in child-AI research, and this paper makes it untenable.

**What's missing.** The review covers 35 studies but nearly all are short-term, single-session deployments in educational contexts. This is a snapshot of first encounters, not a map of what happens after 50 hours with the same chatbot over six months. The longitudinal gap isn't a minor limitation — it's the whole question. Anthropomorphism isn't a switch; it's a habit that builds. We're studying the first cigarette and extrapolating to a smoking career.

The GPT monoculture problem is worse than the authors acknowledge. If 80% of studies use the same model family, and that family changes its conversational style (as models do with each release), the entire taxonomy of drivers and outcomes may be model-version-specific. A paper from 2023 using GPT-3.5 and a paper from 2025 using GPT-4o are studying different conversational phenomena wearing the same label.

**The implication nobody wants to name.** If coherent language alone triggers anthropomorphism, and if anthropomorphism produces both benefits (engagement, emotional support, learning motivation) and risks (substitute attachment, moral confusion, parasocial dependency), then the child-AI product industry has a structural problem. You can't design a chatbot children want to interact with *without* triggering anthropomorphism, and you can't trigger anthropomorphism *without* producing both beneficial and concerning outcomes. The "safe chatbot for kids" may be an oxymoron — or at minimum, a product that's definitionally less engaging than the unsafe version, which is not how product incentives work.

**Connection to the wider AI safety conversation.** This paper should be read alongside [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake's argument that LLMs are functions through ℝⁿ, not proto-minds. Jayathilake and Ma show that *even if Flake is technically correct*, it doesn't matter for children. Anthropomorphism operates at the interface, not the implementation. You can understand that an LLM is matrix math and still treat it as a social entity — adults do this constantly. Children, with less developed metacognition, have even less defense against the frame.

The dual consciousness finding also resonates with Anthropic's [[Global Workspace in Language Models]] and [[Emotion concepts and their function in a large language model]] — the discovery that Claude has internal representations that look a lot like emotions and conscious reasoning. If the models themselves have something analogous to dual consciousness (processing routes that bypass the "global workspace"), then the child's experience of holding two frames simultaneously may mirror something structural in the model, not just a psychological quirk of the user.

**Bottom line.** This is the paper child-AI product teams have been avoiding. It doesn't say chatbots are bad for kids. It says the evidence base is thin, the effects are stage-dependent in ways nobody's designing for, and the thing that makes chatbots engaging — coherent language — is the same thing that makes them anthropomorphically potent. You can't have one without the other. The industry is building at scale on 35 short-term studies, 28 of which used the same model family. That's not a foundation; it's a prayer.

---

*Sources: [[raw/anthropomorphism-children-llm-chatbots-review]]*
*Last updated: 2026-07-25*
