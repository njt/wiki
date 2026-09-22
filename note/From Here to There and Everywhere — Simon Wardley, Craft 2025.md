# From Here to There and Everywhere — Simon Wardley, Craft 2025

Simon Wardley's Craft 2025 keynote runs strategy-as-mapping into an AI-era punchline: maps (anchor, position, evolution axis) beat narrative and magic frameworks; climatic patterns make change anticipatable, from cloud through serverless to AI; and the argument lands on software engineering itself — coding is a craft while testing is the only real engineering we do, so the fix is composable micro tools for coding, with "where do you value humans in the decision-making process" as the most important architectural question of the moment.

---

## The arc

- **Origin (2005).** Clueless Fotango CEO, two Sun Tzu translations from a Charing Cross bookseller, five factors overlaid on Boyd's OODA loop. The realization that triggered everything: he had been "using magic frameworks to run my business."
- **What a map is.** Anchor (the user), position (a chain of needs — value chain within an org, supply chain across orgs), consistency of movement (everything evolves: genesis → custom-built → product → commodity). Mind maps, system diagrams, journey maps: all graphs, all identical under permutation.
- **Patterns.** ~30 climatic patterns (rules of the game), ~40 doctrine (universally useful), ~150 gameplay (market manipulation, e.g. AWS's Innovate-Leverage-Commoditise). Method-matching by evolution stage: XP on the uncharted left, lean in the middle, Six Sigma or outsourcing on the industrialised right — proven at HS2 and Carbon Mapper, and the reason outsourcing contracts fail (they specify the un-specifiable).
- **Change.** Blockbuster died of inertia despite out-innovating everyone; EC2 → DevOps → serverless → FinOps is one repeating pattern since the Industrial Revolution; "no choice over evolution."
- **AI and coding.** Vibe coding is an architectural decision, not a convenience. Testing shows what real engineering looks like; coding doesn't yet. The prescription: composable micro tools built for context, tool first, then the problem.
- **Sovereignty.** Whoever controls tools, medium and language controls how you reason — "new theocracies." Openness (including training data), diversity, critical thinking as the counter; China has the map rooms the West lacks.

## Key quotes

> "Our CEO was completely clueless... I know this for a fact because I was the CEO."

The talk's origin story, and the reason it lands: he isn't diagnosing the audience's leadership, he's confessing his own.

> "The entire multimillion pound project just disappeared in a puff of smoke. These people weren't daft, they were trapped by past narrative."

The insurance robotics story. The bottleneck was real — servers had to be physically modified to fit custom racks — but the map exposed that the *reason* for custom racks (there was no such thing as standard racks once) had evaporated. They were optimizing process flow while the system itself had evolved.

> "There's no such thing as one size fits all methodology... Even SAFe has its context. It's just it doesn't know what it is."

Followed by: "Every time I turn up at an Agile conference and say agile doesn't work everywhere, it's burn the heretic." Method choice is a consequence of evolution stage, not ideology.

> "Blockbuster out innovated everyone. That's not what killed them. Inertia did."

The quiz setup is brutal: Blockbuster was first with the website, first with online ordering, first with streaming experiments — and bankrupt first anyway. Late fees (which required stores) were the revenue their past success protected.

> "This is not an architectural diagram. This is a statement of belief... The code is the architecture... The real decision maker and architect was always the coder."

The hinge of the AI half. If the code is the architecture and the coder is the architect, then handing the coding to an AI *is* handing over architectural decision-making — which makes vibe coding a values question, not a productivity one.

> "Coding is a craft. Testing is engineering... The single most costly thing we do in development, we never discuss."

Tests are hypothesis-driven, systematic, composable micro tools that generate their own information; coding is one monolithic tool everywhere plus ad hoc exploration and gut feel. And the budget fact he hammers: over 50% of development time goes to reading code, which nobody optimizes.

> "Kitchen blenders are good for blending soup, but not for building deep mine shafts... regardless of what the kitchen blender salespeople tell us."

His metaphor for monolithic dev tools — and for the vendors selling AI-augmented versions of them: "They have a lovely multi billion dollar industry. I don't think they do us any favors."

> "We spent the first three weeks building the tool to solve the problem and then we solved the problem."

The feenk/Tudor legacy-translation story: 10–20 person-years of failed effort before them, a CIO betting on failure, and "we're finished" at the four-week meeting. Build the tool first; the problem is then small.

> "You don't get into a supply chain war with somebody who actually understands the supply chain."

The sovereignty coda: Western governments have no map rooms; China is "spectacularly good" at this.

## Themes

#person (Simon Wardley) · #concept (Wardley mapping; evolution axis; climatic patterns) · #pattern (inertia; co-evolution of practice; Jevons explosion; method-matching) · #tool (composable micro tools; the kitchen-blender monolith)

## The take he'd want you to argue with

The talk's rhetorical engine is mapping-as-honesty, and its load-bearing component is the evolution axis. That is also its soft spot: placement on the axis is a judgment call, and the method's accuracy hangs on it. In the insurance story the team mapped "compute as a product," Wardley said commodity, and the transcript records his entire response as "that's okay." Maps depersonalize challenge — "I don't challenge their story, I challenge the map" — but a map is only as good as its placement, and the talk never shows how placement disputes get resolved. Every anecdote is a triumph (£425M on one government project, 70% of cloud for Canonical on half a million pounds, a four-week legacy win); there are no failure modes of mapping itself, no cost of mapping a large estate, no story of a bad map doing damage. That's survivorship-shaped evidence for a method whose whole pitch is seeing clearly.

The craft/engineering distinction is the best thing in the talk and slightly question-begging. Testing is engineering *because we built it that way* — hypothesis first, systematic exploration, generated information, composability. Wardley's claim is that coding could be the same, and the demo (build thousands of contextual tools, then solve the problem) is a real existence proof. But the economics go unexamined: who maintains thousands of contextual tools, who owns them, whether small teams can afford tool-first. The follow-up session apparently demonstrated it; the keynote only asserts it. And "the test suite is the best specification of a system" ignores test rot and coverage theater — the specification is only as good as its maintenance discipline, which is exactly the problem he diagnoses everywhere else.

Where the talk is genuinely ahead of most 2025 commentary: "the most important architectural question today is where do you value humans in the decision-making process" reframes the vibe-coding debate as value placement rather than tool choice, and his "AI hallucinates all the time. It's just that some of the hallucinations we catch out as being wrong and the others we don't" is the correct description of nondeterministic code, not a joke. The sysadmins-returned-as-DevOps-engineers rebuttal to "AI will replace engineers" is historically fair but not a proof; the Jevons-paradox argument — efficiency explodes demand for higher-order systems — is the stronger one. The sovereignty section is the most interesting and least developed: tools-medium-language control reasoning, values baked into training data (trolley problem, gold-member route priority), "the market is basically a bunch of idiots" — raised, then abandoned in four minutes because he'd run out of time. The transcript's own artefacts tell on it: the auto-transcription renders "Wardley mapping" as "worldly mapping," which is either a bug or the most honest description of what a good map does.

## How this connects

- [[Lean Software Production]] — Matt Wynne argues AI makes methodology existential; Wardley supplies the mechanism and the receipts: method-matching by evolution stage delivered HS2 under budget in 2012, and "even SAFe has its context, it just doesn't know what it is" is the sharpest one-line version of the same claim.
- [[The New Software Lifecycle]] — Osmani draws "verification as the line between vibe coding and engineering"; Wardley draws the same line as a map boundary — outsource what you don't care about, vibe-code prototypes, engineer-with-review where it matters — and names it "architecture is an expression of values."
- [[Optimizing for Decision Points]] — Simister designs workflows that surface taste-sensitive decisions; Wardley states the underlying question those workflows should answer: where do you value humans in the decision-making process, the era's most important architectural decision.
- [[The Forest and the Desert Are Parallel Universes]] — Beck (same stage, same conference) shows the desert's gravity; Wardley shows the specific mechanism of a past-success trap: Blockbuster out-innovated everyone and died of late-fee-shaped inertia anyway — inertia is not diffuse resistance but a revenue model defending itself.

---
*Sources: [[raw/from-here-to-there-and-everywhere-simon-wardley-craft-2025]], [[summary/from-here-to-there-and-everywhere-simon-wardley-craft-2025]]*
*Last updated: 2026-09-13*
