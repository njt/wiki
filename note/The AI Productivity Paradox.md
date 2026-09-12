# The AI Productivity Paradox

Marty Cagan diagnoses why AI's acceleration of software output hasn't translated into business outcomes: most teams are using AI to speed up a broken project model rather than switching to a product model that distinguishes discovery from delivery. The piece is sharp, but its confidence that evidence will eventually win masks the real novelty — AI doesn't just amplify the output-over-outcomes flaw, it creates an evaluation bottleneck where even teams that *want* to build to learn can't keep up with what's being built.

---

## Key Quotes

> "89% of executives say AI has increased the speed of work, but only 6% feel confident they can point to specific organization-wide AI ROI."

The Atlassian stat that anchors the piece. The 89% is noise — of course it feels faster. The 6% is the signal, and it's devastating.

> "It's never been faster to build, which means it's never been easier to run 10 times faster in the wrong direction."

Hilary Gridley's line is the sharpest in the piece. The speed multiplier is symmetric — it applies equally to good ideas and bad ones. This is the dynamic that makes the paradox *worse* over time, not self-correcting.

> "AI makes building easier, but the hardest part remains knowing what to build."

Chip Huyen's contribution cuts through the framing debate. The hardest part of product work was never implementation speed. AI changes the supply curve for something that was never the bottleneck.

> "Rather than closing the gap, the strong product companies are increasing the distance between themselves and the majority of the market."

Cagan's honest revision of his own earlier prediction. He thought AI would be an equalizer. It's been the opposite. The mechanism: good product culture compounds with AI velocity; bad product culture just produces bad output faster.

## Key Themes

- **#concept Output vs. Outcomes** — The central distinction. Output is what you build; outcomes are what changes for the customer and the business. The project model is designed to deliver output. AI supercharges output. The paradox dissolves when you realize the model was never designed for outcomes in the first place.

- **#pattern Build to Learn, Build to Earn** — Cagan's product-model distinction. Discovery ("build to learn") tests whether ideas are worth building. Delivery ("build to earn") makes them production-grade. AI accelerates both, but if discovery is skipped, you're just industrializing guesswork.

- **#concept The Product Model** — A mode of working where teams are empowered to solve problems rather than deliver features, measured on outcomes rather than output. Cagan has been advocating this for years; the AI paradox piece is essentially "the product model matters more now, not less."

- **#pattern The Evaluation Bottleneck** — Implicit in Cagan's argument but never named: when AI lets you produce 10× the output, the bottleneck shifts from building to evaluating what was built. Discovery can't keep up with delivery if discovery remains human-paced. This is the genuinely new problem AI introduces — not just faster bad output, but an inability to sort good from bad at AI velocity.

## Critical Analysis

Cagan is right about the diagnosis — the project model was always output-obsessed, and AI is a force multiplier on a broken process — but wrong about the prognosis. His implied theory of change is "leaders will eventually confront the evidence and adopt the product model." There are at least three problems with this:

**1. The evidence has been here for a while and hasn't worked.** Ludicity's field report from ~300 meetings documents an 18-month, 0% AI project success rate — and the organizational response is doubling down, not course correction. The executive prisoner's dilemma around AI investment means honesty is a dominated strategy. [[AI Mania Is Eviscerating Global Decision-Making]] captures the dynamic Cagan is missing: the market isn't punishing output-over-outcome fast enough to create the selection pressure he's counting on.

**2. AI creates an evaluation bottleneck that the product model doesn't solve.** The product model assumes discovery can keep pace with delivery. When a team builds 3 prototypes per quarter, user testing and stakeholder review scale fine. When an AI-augmented team produces 30, discovery becomes the bottleneck — not because the team lacks product culture, but because human evaluation doesn't accelerate at the same rate as AI generation. This is the genuinely novel problem Cagan's pre-AI framework doesn't address: even teams that *want* to build to learn can't evaluate what they're building fast enough.

**3. The strong-get-stronger dynamic is more structural than Cagan admits.** He observes that strong product companies are pulling away, but attributes it to product culture. The simpler explanation is that AI rewards *existing* product sense — the taste, judgment, and customer intuition that comes from years of doing discovery. You can't AI-accelerate your way to taste. The companies that already had it can now execute on it faster; the companies that didn't can't acquire it any faster than before. AI didn't widen the culture gap directly — it just made the consequences of having good product culture more visible, faster.

**What Cagan gets exactly right:** The paradox isn't really a paradox. If you've been measuring output and calling it progress, AI giving you more output was never going to change your trajectory. The 89%/6% gap isn't a mystery — it's a measurement error made visible at scale. And the framing of AI as an amplifier rather than a solution is correct, important, and under-discussed. Ben Evans' [[AI, Tools and Transformation]] sharpens the point from the enterprise side: the hard part was never making the tool, it's knowing you need one — which is why handing everyone the model produces adoption, not transformation.

**Brooks' time-scale lens clarifies the paradox further.** Rodney Brooks distinguishes four time scales for technology: research (10–20+ years), hype (months), at-scale deployment (20+ years), and economic reshaping (50+ years). Cagan's paradox is what happens when an organization confuses Time Scale 2 (AI tools exist and are hyped) with Time Scale 3 (those tools are deployed at scale in genuinely transformed organizations). The tools are real but the deployment hasn't had its two decades yet — and no amount of executive urgency compresses that timeline. [[Four Time Scales for Technology Development and Deployment]]

Tim O'Reilly frames the same problem as the new Solow paradox — "you can see the computer age everywhere except in the productivity statistics" — and argues it disappeared for computers in the late 90s not because computers got faster but because companies reorganized around them. [[AI as an Enterprise Operating System]] provides the most concrete public recipe for that reorganization: capability ladders, hackathons as training infrastructure, curated skills marketplaces, and "scar tissue turned into infrastructure" — organizational redesign as mechanism design, not procurement or communications.

The structural prescription Cagan leaves implicit gets spelled out at the backlog level in [[Backlog Hierarchy Problem]]: its two-backlog principle — an opportunity backlog for discovery, a delivery backlog for committed work — is Cagan's discovery/delivery split made concrete, and its three-tier hierarchy (objectives → initiatives → ideas) is what stops mixed-altitude items from being ranked as peers, the comparison every scoring framework silently assumes away.

## Related Pages

- [[Discovery Debt]] — The accumulated weight of untested assumptions that compounds invisibly until products are expensively wrong
- [[Nicole Forsgren on AI and Developer Productivity]] — Why shipping hasn't gotten faster: the bottleneck shifted from inner loop to outer loop
- [[Laura Tacho — Data vs Hype]] — 92.6% adoption but low transformation: AI as accelerator not fixer
- [[Writing Code vs. Shipping Code]] — 180% AI-driven commit gains attenuate to 30% at release level
- [[Five Studies That Are Changing How I Think About AI in Software Engineering]] — The productivity-experience paradox and cognitive/intent debt
- [[AI Mania Is Eviscerating Global Decision-Making]] — 0% AI project success rate over 18 months, the executive prisoner's dilemma
- [[Product-Minded Engineers in an AI-Native World]] — Product engineering as motivation not role, taste as trainable craft
- [[Building World-Class Engineering Teams in the Age of AI]] — Bottleneck migration left and right of code
- [[The New Software Lifecycle]] — AI unevenly compresses the SDLC; harness over model
- [[Hidden Inefficiencies Behind Delivery Delays]] — Five invisible delivery killers with measurable indicators
- [[Dev Machine Foundry]] — Priority inversion: measurable work crowds out valuable work
- [[Building When It Feels Like There's Nothing Left to Build]] — Chip Huyen on why build at all when AI can build anything

---

*Sources: [[raw/ai-productivity-paradox]]*
*Last updated: 2026-07-25*
