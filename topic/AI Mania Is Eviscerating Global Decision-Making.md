# AI Mania Is Eviscerating Global Decision-Making

Ludicity's field report from ~300 meetings with organizations worldwide documents a mass psychosis: AI investment is near-universally failing, yet honest discussion of that failure is professionally fatal. The essay is simultaneously a diagnosis of the coordination trap executives are caught in, a survival guide for people trapped inside it, and the most darkly funny account of corporate AI dysfunction yet written.

---

## Key Quotes

> "Almost every report at a company about 'massive AI productivity gains' is untrue as a matter of brute fact."

The central empirical claim, backed by an 18-month, 0%-success-rate observation window across the author's client engagements. Not "AI doesn't work" — the author grants that AI tooling can accelerate specific workloads. The claim is narrower and more damning: the *investment pattern* is senseless regardless of tool capability. Companies can't run software projects competently, and AI projects carry every normal failure mode plus novelty risk. The 0% figure is the essay's load-bearing beam — if you don't believe it, the rest reads as cynicism; if you do, the rest reads as documentation.

> "Everyone has learned very quickly to praise executives on their visionary AI prowess, or they will be gunned down in the proverbial streets."

The essay's most quoted line captures the dynamic that makes the situation self-reinforcing. Dissent is punished, AI-washing is rewarded, and the feedback loop between honest information and leadership decisions has been severed. This isn't ordinary corporate politics — it's a dynamic where the *only* people fired over AI are skeptics, never true believers, regardless of results.

> "Doctors don't walk around showing off cool pills that they'd never prescribe."

After watching lukewarm clients demand to buy an AI product the author's team had just finished explaining wouldn't work — having been warned, repeatedly, that it achieved ~92% accuracy and would give their CFO one wrong number in ten — the team removed AI from their demos entirely. This is the essay in miniature: the rational evaluation system shorts out when AI enters the room.

> "Executives around the world nervously pointing guns at each other."

The game-theoretic diagnosis: CEO A can't contradict CEO B's 100x productivity claims because CEO B's company is a customer. CEO C can't contradict CEO A because A is C's board member. The result is a prisoner's dilemma played across the executive class, where universal defection (lying about AI) is individually rational and collectively catastrophic. Every executive privately knows it's nonsense; every executive publicly doubles down. The chain of mutual blackmail means no one can be the first to say the emperor is naked.

> "Just lie. It's fine. History will forgive you. Save that puppy."

The most startling piece of survival advice: if you need resources for something important and the only way to get them is to add a $10,000 AI chatbot to the budget that you know will do nothing — do it. This isn't cynicism; it's triage. The author's moral framework here is consequentialist: the thing that actually helps people matters more than honesty about a line item that exists only to satisfy a purity test.

> "Almost every large organisation that I am aware of is no longer able to focus on anything important."

The essay's darkest conclusion. The AI-native purity test — must demonstrate AI usage, must allocate budget to AI, must frame every initiative as AI-driven — has become a distraction mechanism that prevents organizations from doing their actual work. This is the damage that will persist after the bubble pops.

---

## Key Themes

- **#pattern AI-Washing** — Doing real work and claiming AI did it, because managers demand AI usage regardless of results. The Go→Zig translation charade is the canonical example: an engineer runs a pointless AI task in the background to satisfy usage tracking, then does their actual job normally. The AI-washing dynamic is distinct from greenwashing because it's not about external reputation — it's about internal survival.

- **#pattern The Coordination Trap** — The executive prisoner's dilemma where mutual defection (everyone lies about AI) is individually rational and collectively catastrophic. This is the essay's most novel contribution: it's not just that executives are stupid or greedy; they're trapped in a game-theoretic structure where honesty is a dominated strategy. This distinguishes Ludicity's analysis from mere cynicism.

- **#concept AI-Native Purity Tests** — Funding, headcount, and career advancement gated on demonstrating AI usage. Projects that can't easily be labeled "AI" starve. Projects that are standard work with an AI sticker thrive. The database migration example — hand-translated SQL billed as AI-driven — is the pure case. The purity test doesn't improve outcomes; it distorts resource allocation.

- **#concept True Believers vs. Cynics** — The essay's crucial distinction. Cynics who lie about AI for careerist reasons can potentially be reasoned with. True believers — including executives who've never used ChatGPT but produce AI strategies — are "impervious to even inducement by self-interest." The scariest finding isn't that people are lying; it's that many have stopped knowing they're lying.

- **#pattern The Mind-Killer Demo** — Showing someone an AI demo overrides their rational evaluation. The Snowflake Cortex story is the definitive account: clients explicitly told "this won't work for you" still demanded to buy it. The mechanism isn't deception — the author's team was honest about the limitations. It's something closer to possession.

---

## Critical Analysis

**This is the best piece of writing on corporate AI dysfunction I have read.** It earns its authority the hard way: ~300 meetings, direct observation, specific examples with named companies and products. The essay never reaches beyond its evidence. When the author makes a claim about 0% success rates, they specify the observation window (18 months) and scope (their engagements). This is what separates it from the endless "AI is a bubble" opinion pieces that float through tech media — it's grounded fieldwork, not vibes.

**The game-theoretic frame is the essay's real contribution.** "Executives are lying about AI" is an observation anyone could make. "Executives are trapped in a multi-party prisoner's dilemma where honesty is a dominated strategy, and here is the exact mechanism" — that's analysis. The chain-of-mutual-blackmail model (CEO A → CEO B → CEO C → CEO A) is simple enough to be convincing and specific enough to be falsifiable. It also explains why the bubble persists despite widespread private skepticism: no one can be first.

**The survival advice is surprisingly practical and surprisingly bleak.** "Assume the organization will burn you out and fire you. Start looking as if you've already been fired." "If your manager sends AI-generated text, respond with AI-generated text — they actually like it." These aren't theoretical positions; they're field-tested tactics from someone who's been inside the machine. The advice to contractors — "you'll be paid more and mostly left out of internal politics" — is the quietest and most damning line in the essay.

**The 0% claim demands a caveat.** The author's team rejected all AI implementation work. Their observation is of projects they declined to participate in plus projects they observed in passing. This is selection bias in the literal sense — they selected themselves out of AI work. But it's selection bias that cuts against their interests (they turned down money), which makes it more credible, not less. A team that *was* doing AI implementation and reporting 0% success would be a stronger claim. What we have is still strong: a team that looked at enough projects to conclude the pattern was universal enough to justify walking away from revenue.

**What's missing: the counterexamples.** The essay doesn't engage with organizations where AI investment *is* working — Anthropic shipping 65% of product code through Claude Tag, Cursor's growth, the companies where [[Running an AI-Native Engineering Org]] documents real transformation. The author would likely respond that these are the exceptions that prove the rule, and that the essay's scope is explicitly the "large organisations" where dysfunction reigns. Fair enough. But the line between "this is what I've seen" and "this is universal" is one the essay rides hard, and a reader should know which side they're on.

**The essay pairs with** [[AI Will Not Make You Rich]] (AI's containerization thesis — value flows to customers, not builders), [[Laura Tacho — Data vs Hype]] (92.6% adoption but low transformation — the data that backs Ludicity's anecdotes), [[The Dead Economy Theory]] (productive capacity without human participation), and [[Why We Fear AI]] (AI anxiety as capitalism anxiety — Ludicity's account is the capitalism anxiety made concrete). It's the dark mirror of [[Writing Code vs. Shipping Code]]: if 180% AI commit gains attenuate to 30% at release, what does that say about the CEO claiming 100x? It also pairs with [[The Cult of Vibe Coding Is Insane]] — Ludicity is documenting the same cult dynamics Bram Cohen identified, but at the executive rather than the engineering level.

Rodney Brooks provides the structural explanation for what Ludicity observes: Time Scale 2 (hype generation) is a distinct phase with its own dynamics — technologies erupt from obscurity to daily business-press coverage in months, everyone re-brands their existing work, and then it dies down when the next hype arrives. Ludicity's ~300 meetings capture the moment when an entire executive class is caught inside a Time Scale 2 bubble and cannot exit because the prisoner's dilemma makes honesty a dominated strategy. The graveyard Brooks catalogs (blockchain, metaverse, Watson, nanotech, expert systems) is where today's AI mania will eventually rest — but not before it does real damage to the organizations trapped inside it. [[Four Time Scales for Technology Development and Deployment]]

**Read it alongside** [[Human-in-the-Loop is Tired]] for the psychological cost from the engineer's perspective, [[The Education of the Broligarchy]] for the ideological underpinnings of the true-believer executive, and [[The Flat Curve Society]] for Steve Yegge's complementary diagnosis of the AI plateau.

---

*Sources: [[raw/ai-mania-eviscerating-decision-making]]*
*Last updated: 2026-07-21*
