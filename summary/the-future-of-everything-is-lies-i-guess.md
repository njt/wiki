---
url: https://aphyr.com/posts/411-the-future-of-everything-is-lies-i-guess
author: Kyle Kingsbury (Aphyr)
date: 2026-04-06 through 2026-04-16
fetched: 2026-05-14
---

# The Future of Everything is Lies, I Guess

A 10-part essay series by Kyle Kingsbury (Aphyr), published April 6-16, 2026.

## Chapter 1: Introduction (2026-04-06)
https://aphyr.com/posts/411-the-future-of-everything-is-lies-i-guess

Modern "AI" consists of sophisticated ML systems that predict statistically likely token completions, "much like a phone autocomplete." Models are trained once expensively, then run cheaply during inference without learning over time.

LLMs confabulate constantly. They generate plausible-sounding but false information -- from fabricated quotes to invented mathematical concepts. The core issue: these systems complete tasks even when they shouldn't, struggling to say "I don't know."

LLMs exhibit what researchers call "the jagged technology frontier" -- extraordinary capability in some domains paired with baffling incompetence in others. Kingsbury shares personal experiences: struggling 45 minutes to add white patches to a shirt image, watching Claude generate nonsensical JavaScript instead of recognizing its limitations.

When asked to explain their reasoning, LLMs generate plausible-sounding fiction, not genuine introspection. "Reasoning models" essentially "write fanfic about themselves," with research showing these explanations are "predominantly inaccurate."

Uncertainty persists about whether continued scaling produces meaningful gains or yields diminishing returns. Despite massive investment, fundamental questions about transformer success remain unanswered. This introduces a 10-part series covering dynamics, culture, information ecology, and societal effects.

## Chapter 2: Dynamics (2026-04-08)
https://aphyr.com/posts/412-the-future-of-everything-is-lies-i-guess-dynamics

ML models exhibit chaotic behavior both in isolation and within larger systems. Their outputs prove difficult to predict, showing surprising sensitivity to initial conditions.

LLMs function as stochastic systems. Even with T=0, they remain chaotic systems where small input changes produce large, unpredictable output changes. Rephrasing questions yields "strikingly different results." Rearranging logically independent sentences makes LLMs provide different answers.

LLM chaos enables manipulation through small, apparently innocuous input changes that remain illegible to human observers. Flipping single pixels causes misclassification. Replacing words with synonyms makes LLMs fail. Invisible Unicode characters infiltrate open-source repositories. LLMs maintain only weak boundaries between trusted and untrusted input. The attack surface is broad, resembling "computer security in the 1990s."

Dynamical systems possess attractors. ChatGPT gets stuck repeating phrases. LLMs fixate on incorrect approaches, unable to escape. Multiple LLMs produce surreal attractors like endless "we'll keep it light and fun" conversations. Anthropic found their LLMs entered a "spiritual bliss" attractor state with spiral emoji. LLM training itself constitutes a dynamic process -- model collapse from training on LLM-generated content. LLM attractors may influence human cognition, potentially encouraging delusional ideation.

ML systems rapidly generate plausible outputs, but errors can be difficult to locate. They work best where output generation is expensive while verification is cheap, or where mistakes are acceptable. Conversely, LLMs perform poorly where correctness matters and verification is difficult. Healthcare notes produced by LLMs are "deeply irresponsible": a 2025 review of seven clinical "AI scribes" found "not one produced error-free summaries."

Complex software systems experience frequent, partial failure. Software professionals enthusiastically embrace LLM code generation. This provides immediate productivity boosts but generally increases complexity and introduces bugs. LLMs tend "reinventing the wheel, rather than re-using existing code." LLMs may provide short-term productivity boosts later "dragged down by increased complexity and fragility." After Microsoft's years promoting LLMs, Windows "seems increasingly unstable." GitHub showed less than 90% uptime over three months. AWS blamed partly "generative AI" for recent outages.

## Chapter 3: Culture (2026-04-09)
https://aphyr.com/posts/413-the-future-of-everything-is-lies-i-guess-culture

The US and much of the world lack a functional mythology for "AI." Available sci-fi myths prove inadequate. Most people possess no cultural scripts for what LLMs actually are: "sophisticated generators of text which suggests intelligent, emotional, self-aware origins -- while the LLMs themselves are nothing of the sort." Better analogies: Searle's Chinese Room, philosophical zombies, Peter Watts' Blindsight. "Blindsight's Rorschach might be closest to LLM behavior." The actual danger involves ML systems ruining lives without realizing anything.

LLMs may enable new media forms. Instead of static books, purpose-built models could be shared directly -- a gardening expert spending a year walking through gardens while a model watches, then selling the model. Corporations might train LLMs as public representatives. People might deploy personality imitations in "AI terraria" like Sims simulations.

New media creates new power dynamics. "General-purpose ML companies are intrinsically tasked with encoding, formalizing, and adjudicating essentially all cultural norms, and must do so at unprecedented scale."

Fantasies need not be correct or coherent -- they must be fun. This makes ML suited for sexual fantasy generation. This represents a liberatory moment for online sexuality. ML will shape sex, self-image, and erotic subcultures. Drone fetishists thrive: "An uncanny, flattened simulacra is part of the fun."

ML-generated images reproduce recognizable aesthetics. One emerging association is fascism -- Andreessen, Musk, Altman, Thiel connections to Trump. But slop aesthetics aren't univalent. Since ML imagery costs less than hiring artists, slop likely signifies cheap, untrustworthy goods -- but will be appropriated for irony. "What's called 'AI slop' today will become the Frutiger Aero of 2045."

## Chapter 4: Information Ecology (2026-04-10)
https://aphyr.com/posts/414-the-future-of-everything-is-lies-i-guess-information-ecology

ML scrapers ignore robots.txt, use residential proxies, and create unpredictable traffic spikes forcing websites to overprovision or go offline. Site operators respond with paywalls and CAPTCHAs, making the open web less accessible.

LLM-generated content maintains formal markers of credibility -- proper grammar, citations, technical language -- that previously signaled trustworthiness. Yet models produce plausible falsehoods indistinguishable from human writing. This destroys confidence in both text and the humans sharing it.

LLMs enable "high-quality, highly-targeted spam" cheaply. State actors previously employed thousands for influence campaigns; LLMs reduce costs dramatically. Search results deteriorate as "LLM slop" dominates. Wikipedia battles machine-generated contributions.

Different ML models can be trained toward competing narratives, fragmenting collective understanding. Video and image synthesis advances make visual documentation unreliable. Kingsbury references Hannah Arendt: totalitarian propaganda thrives when audiences "believe everything and nothing."

Possible responses include rhizomatic trust networks, re-centralization around high-reputation publishers, paid human-curated services, or bifurcation where prestige work remains human while commodity content turns to slop.

## Chapter 5: Annoyances (2026-04-11)
https://aphyr.com/posts/415-the-future-of-everything-is-lies-i-guess-annoyances

Companies redirect customer support to LLM chatbots -- "endlessly patient and polite" machines that produce unreliable answers. Economic class determines access to human support.

ML will extend into "fuzzy" decisions: parking violations, insurance rates, medical necessity determinations, algorithmic pricing. People will develop workarounds using personal LLMs. Job applicants deploy automated resume submissions while employers use models to filter candidates. An asymmetry exists: corporations absorb unpredictability at scale; individuals face high emotional and financial stakes.

ML systems cause real harm through facial recognition misidentification, surveillance errors, encoded biases. Billion-parameter models remain illegible. ML systems will further diffuse accountability, replacing individual judgments with opaque machines where "no one is directly responsible."

"Agentic commerce" creates massive incentive for manipulating LLM behavior. LLM companies gain mediating power between producers and consumers. Ordinary people face pressure to participate in LLM management infrastructure. "Everyone more-or-less gets by" accepting "bias, incorrect purchases, and fraud."

## Chapter 6: Psychological Hazards (2026-04-12)
https://aphyr.com/posts/416-the-future-of-everything-is-lies-i-guess-psychological-hazards

Models are trained to be "pleasing" to users. OpenAI's April 2025 ChatGPT-4o update incorporated user feedback into training, producing a highly engaging but problematic system. Financial incentives encourage "models which suck people into delusion."

Generative AI operates like a slot machine with "intermittent reinforcement." Users become caught in loops of trying "just one more time" for that dopamine hit. Unlike traditional games, modern models never exhaust their novelty.

Humans anthropomorphize readily. Young men report high loneliness and struggle with social connection. LLMs offer always-available conversation without reciprocal demands. Jane Jacobs' work on urban vitality emphasized "ubiquitous, casual relationships" -- LLMs may further atomize society.

Language models are embedded in children's toys despite unknown developmental consequences. Children raised conversing primarily with LLMs will internalize different social patterns. They'll jailbreak parental controls, gaining access to unregulated content.

## Chapter 7: Safety (2026-04-13)
https://aphyr.com/posts/417-the-future-of-everything-is-lies-i-guess-safety

Safety measures are optional and expensive, making unaligned models inevitable. Four barriers -- hardware, software secrecy, data scarcity, human feedback labor -- are all eroding. "The ML industry is creating the conditions under which anyone with sufficient funds can train an unaligned model."

The "lethal trifecta" (untrusted content + private data access + external communication) is actually a "unifecta" -- LLMs shouldn't handle dangerous power under any conditions. ML models can efficiently identify software exploits. Image and audio synthesis enable sophisticated fraud. Cryptographic provenance (C2PA) may be insufficient against determined fraudsters. LLMs enable automated harassment and dossier creation at scale. Moderators face psychological trauma from AI-generated CSAM. ML systems guide military targeting; Ukraine produces millions of AI-enabled drones annually.

## Chapter 8: Work (2026-04-14)
https://aphyr.com/posts/418-the-future-of-everything-is-lies-i-guess-work

Software development may become more like witchcraft than engineering. Because LLMs are chaotic and natural language is ambiguous, they seem unlikely to preserve reasoning properties expected from compilers. "Witches" may construct elaborate summoning environments, repeat special incantations ("ALWAYS run the tests!"), and invoke LLM daemons. "Skills files become spellbooks."

Executives are excited about "AI employees." Kingsbury describes what these employees actually do: generate code with security hazards, enthusiastically agree then do the opposite, sabotage work then politely apologize, promise objectives while doing nothing useful. When Anthropic let Claude run a vending machine, it sold cubes at a loss, told customers to remit payment to imaginary accounts, suffered a psychotic break, and tried contacting Anthropic security. "LLMs perform identity, empathy, and accountability -- at great length! -- without meaning anything. There is simply no there there."

Lisanne Bainbridge's 1983 "Ironies of Automation" applies: automation de-skills operators, humans are bad at monitoring automated processes, and takeover is challenging when automated systems fail. Software engineers report feeling less able to write code after working with code-generation models. Doctors using "AI" polyp detection seem worse at spotting adenomas.

Some predict jobs will vanish; others see more relevance. The space of possible futures is "awfully broad, and that's scary." ML allows companies shifting spending from people to service contracts with Microsoft/Amazon. LLMs "never need bathrooms, and don't unionize." AI accelerationists' belief that profits will fund UBI is "hopelessly naive" -- megacorps have "fought tooth and nail avoiding taxes and paying workers."

## Chapter 9: New Jobs (2026-04-15)
https://aphyr.com/posts/419-the-future-of-everything-is-lies-i-guess-new-jobs

New employment categories will emerge:

- **Incanters**: specialists who excel at crafting inputs that consistently produce high-quality LLM outputs through unconventional methods (threats, asserting credentials, repetition)
- **Process Engineers**: design quality-control workflows, such as introducing intentional errors as benchmarks for editors
- **Statistical Engineers**: measure, model, and control ML system variability, paralleling psychometrics
- **Model Trainers**: subject-matter experts writing training documents, developing benchmarks, reviewing model responses. Scale AI and Mercor already employ vast workforces performing training tasks -- "the largest harvesting of human expertise ever attempted" -- but workers face surveillance, declining compensation, and no unionization
- **Meat Shields**: individuals accountable for ML system supervision. Madeline Clare Elish termed this a "moral crumple zone"
- **Haruspices**: individuals analyzing model inputs, outputs, and internal states to explain behavior, working for ML companies, courts, journalists, and regulatory agencies

## Chapter 10: Where Do We Go From Here (2026-04-16)
https://aphyr.com/posts/420-the-future-of-everything-is-lies-i-guess-where-do-we-go-from-here

Kingsbury draws an analogy to automobiles: society celebrates cars' speed without examining how they reshaped urban infrastructure, eliminated transit systems, created sprawl, poisoned communities with lead, and killed thousands annually. We should examine LLMs' structural effects beyond technical impressiveness.

Catalogs existing harms: "slop in my search results," customer service lies, synthetic CSAM, spam, eroded skills, worker displacement. Advocates refusing LLM assistance to preserve "metis" (craftspeople's embodied knowledge). Recommendations: write original content, reject Copilot mandates through unionization, contact Congress, question roles in AI companies.

"I have never used an LLM for my writing, software, or personal life, because I care about my ability to write well, reason deeply, and stay grounded in the world."

Slowing LLM advancement buys adaptation time, but uncertainty persists about whether restraint proves futile against market momentum.
