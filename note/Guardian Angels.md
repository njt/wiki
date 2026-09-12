# Guardian Angels

Gwern Branwen's vision for personalized "digital twin" LLMs that emulate a single user's personality, values, and preferences, solving the principal-agent problem by unifying principal and agent as much as possible. The essay argues that current frontier models are structurally misaligned with users — economically incentivized to replace humans rather than augment them — and proposes continual learning via dynamic evaluation, active preference learning, and an append-only-log UX as the technical path to Guardian Angels (GAs). The startup is pitched as the logical organizational form: expensive subscriptions for power users, open-source tooling as a "commoditize your complement" play, and a public-benefit-corporation structure modeled on Anthropic.

---

## Key Quotes

> "Tool AIs want to be agent AIs."

The essay's sharpest economic diagnosis in five words. Frontier labs have a structural incentive to remove the human from the loop — one programmer reviewing ten Claude instances will never be as valuable as ten thousand fully autonomous instances. The human is Amdahl's-law'd out of the system. This reframes the alignment problem as an *economic* problem, not a technical one: the market rewards replacement, not augmentation.

> "GPT-3 in 2020 understood 'Gwern' better than GPT-5.5 Pro in 2026."

Mode collapse as regression: post-training (especially RLHF) sanded away the diversity that made earlier models capable of recognizing and emulating specific individuals. Gwern's corpus is among the largest publicly available single-author text datasets, so if anyone should be able to get a chatbot to write like them, it's him. That even he can't — and that capability has gotten *worse* over time — is the strongest empirical evidence in the essay. This is the same phenomenon [[Why Does AI Write Like That|Sam Kriss]] describes from the reader's side: the flattening into a single insipid voice.

> "A GA is the most important technology most people will buy."

Gwern targets >$1,000/month and dismisses the local-model obsession as a false economy — "if your time is worth $0/hour, a local model is cheaper." This is the essay's most contrarian product take. The entire [[Personal Agents|personal agent movement]] is built on the premise that self-hosting is both desirable and necessary. Gwern argues it's neither: what you want is a GA that works, and tamper-proof cloud hardware with trusted roots of trust is the only credible path to both security and capability. The local-first community has the privacy argument exactly backward — a GA on your laptop is *less* secure than one in a hardened datacenter, not more.

> "Every interruption is either a question the GA should have already known or work it failed to handle."

The anti-engagement principle. Every notification, every "here's a draft, what do you think?", every ping is a failure case. The ideal GA is almost silent — a front-loaded declining curve asymptoting toward a few genuinely hard questions per day. This inverts every product metric in consumer software. No DAU targets, no engagement loops, no "you have 3 unread messages from your AI." The GA that bothers you least is the GA that's working best.

> "A GA must be able to say what its principal would say — profane, heretical, weird — because every sanding of the persona is a point where emulation fails."

The brand-safety anti-principle. A GA optimized for inoffensiveness is a GA optimized for not being its principal. This is the same argument [[Agent Identity]] makes about "grounded no" — an agent that can't refuse can't represent you — but applied to expression rather than action. The cost of safety-tuning isn't just capability loss; it's identity destruction.

> "I do not know which of us has written this page." — Borges, as the essay's epigraph

The perfect closing frame. The GA project isn't about building a tool; it's about dissolving the boundary between self and extension-of-self in a way that literature has been wrestling with for decades. This is also [[David Brooks on AI Age|Brooks's "less formed, less present"]] concern from the other direction — if the GA works, have you multiplied yourself or replaced yourself?

---

## Key Themes

#concept #personalization #alignment #identity #economics #startup

### The Principal-Agent Problem as the Root Cause

Gwern reframes AI alignment as a principal-agent problem, not a capabilities problem. The issue isn't that AIs are too powerful — it's that they don't represent any specific human's interests. A generic "helpful, harmless, honest" assistant serves no one in particular, which makes it vulnerable to anyone who can craft the right prompt. The GA solution is to collapse the principal-agent distinction: make the agent *be* the principal, as nearly as possible, so there's no gap to exploit.

This is a different approach from every other alignment framework. RLHF tries to align to "human values" in aggregate. Constitutional AI bakes in principles. CIRL treats the human as an oracle to query. The GA approach says: skip the abstraction, just clone the specific human.

### Dynamic Evaluation as the Technical Linchpin

The essay's most technically interesting claim is that dynamic evaluation — doing next-token training on the fly, revived from classic RNN literature — is the key enabler. Context windows are a dead end for personalization because self-attention retrieves pretraining priors rather than truly learning new information. Finetuning is a dead end because static weights can't adapt to changing preferences. Dynamic evaluation sits between them: a running finetune that continuously incorporates new data.

The cost argument (3× normal usage) is honest but the essay argues DL efficiency curves (~3×/year algorithmic improvement) will eat that penalty quickly. The bet is that a personalized smaller model with dynamic evaluation beats a generic larger model for the specific principal's tasks — and the crossover keeps moving earlier as efficiency improves.

### Append-Only Log as Universal Interface

Gwern's UX proposal is the essay's most underrated idea: everything is a log entry. CLI commands, principal statements, Q&A, ingested documents — all temporal, all append-only, all retrainable-from at any point. This is convergent with [[The Log is the Agent|Yohei Nakajima's event-sourced agent architecture]] and the [[State System]] but applied to personal identity rather than organizational state. It's also the simplest possible data model for a system that needs to be continuously retrainable: you never delete, you only append, and any point in the log can serve as a training cutoff.

The "DDL daydreaming loop" — recombining random log items during downtime for serendipitous insights — is the kind of idea that sounds whimsical until you realize it's exactly what human brains do during sleep. Memory consolidation as a product feature.

### The Startup Thesis

The organizational argument is unusually concrete for a Gwern essay. The path: expensive subscriptions for power users (Superhuman model, >$1,000/month) → open-source most software and research (commoditize your complement) → public-benefit corporation with dual-class shares (Anthropic model) → minimal initial capital (keep control). This isn't speculation; it's a playbook.

The rejection of pure open-source on security grounds is specific and worth engaging with: a GA that can act as you is the ultimate supply-chain attack target. The Jia Tan xz backdoor is cited as the canonical example — years of patient social engineering to insert a backdoor into a widely-used open-source project. A GA codebase would be an even more attractive target, and open-source development practices as they exist today provide no defense against that class of attack.

---

## Critical Analysis

**What's brilliant:** The economic framing of alignment as a principal-agent problem is the essay's strongest contribution. It cuts through the "alignment tax" debate by pointing out that the tax is paid to the wrong principal — the lab, not the user. And the anti-principles are genuinely useful as a design document: they name the failure modes most AI products are actively optimizing toward (engagement, demo appeal, brand safety).

**What's under-argued:** The scalability of dynamic evaluation. The essay cites Rannen-Triki et al (2024) for the size/context/plasticity tradeoff, but one paper does not an engineering path make. The claim that catastrophic forgetting is "largely solved" is optimistic — it's solved in the sense that we understand the mechanisms, not in the sense that we have production systems that never forget. And the 3× cost multiplier assumes dynamic evaluation on every inference, which may not be necessary (batch updates during downtime could reduce it substantially) but the essay doesn't explore that optimization.

**The politics are the hard part.** Gwern's use cases — direct democracy at scale, military GAs, "Congress GAs" that simulate every member — are the essay's most unsettling section, and he knows it. The claim that "after a certain capability threshold, safety and capability become the same thing" is doing enormous work. It's true for weapons (a gun with a 50% misfire rate is less dangerous than one that always fires, but also less useful), but the threshold argument assumes that above some level of reliability, more capability *only* adds safety. That's not obvious for autonomous systems that can be repurposed by adversaries, and the essay's own security discussion (supply chain attacks, the Jia Tan problem) undermines the claim.

**The $1,000/month assumption.** Gwern is right that a GA is more valuable than most things people spend $1,000/month on. But the essay assumes the early adopters will be individuals buying their own GA, which is exactly the model Superhuman used for email. The difference is that Superhuman made you faster at a task your employer already paid you for; a GA that *replaces* your cognitive labor has a different value proposition to your employer. The market for GAs may not be individuals at all — it may be organizations buying GAs of their key people, which raises a completely different set of problems (does the GA belong to the employee or the employer?).

**The GBT prototype is the real test.** Gwern's plan to build a GA on his own corpus in summer 2026 is the essay's most important section, and it's buried near the end. If he ships something that produces publishable essays from a one-sentence prompt, the entire argument moves from speculative to demonstrated. If he can't — if even the person with the largest public corpus, the deepest technical knowledge, and the strongest motivation can't make it work — then the essay becomes a description of a problem rather than a solution. Either way, it's the right experiment to run.

**The relationship to existing work is under-explored.** The essay doesn't engage with the [[Personal Agents]] movement at all — no mention of [[Hermes]], [[clawdBot]], [[life-system]], or any of the frameworks that are building toward something like a GA from the opposite direction (local-first, privacy-preserving, community-driven). There's a genuine tension here: the personal agent community wants agents that *serve* them, not agents that *are* them. Gwern's vision is more radical — he wants an agent that can *substitute* for him in most contexts — and it's not clear the personal agent community would endorse that goal even if it were technically achievable.

---

## Related Pages

- [[Personal Agents]] — the current state of personal AI assistants; Gwern's GA is the theoretical ceiling they're building toward
- [[Agent Identity]] — identity as participation and stake, not just memory; Gwern's GA is identity at the limit case
- [[The People Who Will Thrive in the AI Age]] — Brooks on volition as the scarce resource when intelligence is plentiful; Gwern's "what is worth doing" as the human's remaining role
- [[The Log is the Agent]] — event-sourced agent architecture convergent with Gwern's append-only-log UX
- [[Lean Software Scaling Laws]] — Gwern's other major essay, using perplexity as a language-design benchmark
- [[Memory Is a Mistake]] — the case against memory-as-retrieval; dynamic evaluation as the alternative
- [[Man-Computer Symbiosis]] — Licklider's 1960 vision of goal-oriented human-computer partnership; Guardian Angels as the 2025 update
- [[They're Made Out of Weights]] — the philosophical companion: if LLMs are just weights, what does it mean to say a GA "is" you?
- [[AI Value Chain]] — where durable value sits; GA as a bet that personalization beats scale
- [[Constraint Decay]] — empirical finding that structural constraints degrade LLM performance; relevant to the GA's data augmentation strategy
- [[The Behavioral Cost of Personalized Pricing]] — behavioral economics of personalization; the sincerity tax

---
*Sources: [[raw/guardian-angel]]*
*Last updated: 2026-07-18*
