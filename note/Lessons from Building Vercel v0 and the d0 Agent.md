# Lessons from Building Vercel v0 and the d0 Agent

Malte Ubl (CTO, Vercel) on building agents in-house — both the internal data agent Dzero and the V0 product agent. The talk's central provocation: building agents is "extremely easy" and you don't need to buy them. Simplicity scales as models improve, framing problems as coding tasks unlocks disproportionate capability, and "teams just make things go slower." Delivered at [[The Pragmatic Summit]].

---

## Key Quotes

> "In the world of agents you have to be humble in the sense of we're just discovering how to build them. Just because something was best practice like the summer of 2025 means quite little today."

Ubl deleted Dzero's first version — a multi-tool loop architecture — and rebuilt it as two tools: bash and Execute SQL. ~50 lines of code. The YAML semantic layer (plain-English descriptions of every Snowflake column) is all the agent needs to grep through and write queries. This is a bet that models will get smarter, not that you need to get smarter about prompting them. The humility is tactical: don't over-invest in architecture that models will obsolete.

> "If you can make things look like they're a coding task, even though they're not, then you get disproportional good results."

The most transferable technical insight in the talk. Models were trained heavily on code — exploit that. Text-to-SQL becomes a bash script that greps a YAML file. CSS layout becomes inline Tailwind classes. The framing matters more than the problem domain. This is the same pattern behind [[Webwright]] (browser automation as Playwright code) and [[Tone LLM]] (LLM fills a JSON schema, deterministic code does the rest). The agent doesn't need to understand your world — it needs to see your world as code.

> "I'm deep in 12 hours a day coding. And by coding I mean I can do meetings."

Ubl's current stack: Claude with 4.6 fast and Codex 5.3 for code reviews. The CTO of Vercel describes his own workflow as coding *through* meetings — the agent works while he talks. This isn't a demo; it's how a shipping product's CTO operates day to day. The collapse of "coding" and "managing" into one continuous activity.

> "Teams just, I don't know, make things go slower."

The full prescription: one person finds product-market fit and produces a working demo *first*, then you allocate a team. Premature team formation is organizational drag, not leverage. This echoes the [[The Founder's Playbook]] stage model (Idea → MVP before scaling) and [[UBTRIPPIN Dispatches]] (one AI-augmented individual building a startup). The counterargument is that solo exploration works for *discovery* but not for *hardening* — Ubl acknowledges this implicitly by saying "if the demo is good, I'm going to give you a team."

> "There are no approvals. Anyone can ship anything, but they have to tell the organization that they're going to do it and the organization can veto things."

Ubl calls this "optimistic locking." Legal doesn't have to say yes (which creates waiting) — they must actively say no. This inverts the default from "blocked until approved" to "shipping unless stopped." The organizational cost is that vetoers must be vigilant, and veto failures are public. This only works if the culture actually vetoes things — if veto becomes taboo, optimistic locking degrades to no locking at all.

> "We automated 87% by now of our support intake" — support agents now "have a much better job" handling only hard problems.

Automation didn't eliminate support roles; it upgraded them. The remaining 13% are the genuinely hard problems. Support agents went from ticket triage to complex problem-solving. This is the optimistic version of the automation story — the pessimistic version is that the 87% was the training ground for the skills needed to handle the 13%, and you've broken the career ladder. Ubl doesn't address this.

> "Software is free, like getting a free puppy. It has to be maintained."

The best line of the talk, and not original to Ubl — but he deploys it perfectly. The marginal cost of creating software goes to zero with agents, but the maintenance burden doesn't. Every generated app is a puppy you now have to feed. This is the same tension in [[The solution might be cancelling my AI subscription (Willison)]] — even good AI-generated code creates maintenance obligations faster than you can meet them.

> "The most senior ICs are in a way the one that benefit the most because they... now have just more minions."

Senior ICs already orchestrate work — delegating to agents is a natural extension. Junior engineers benefit as "digital natives" who never learned the old way. "What's most interesting is kind of what happens in the middle." This is the most honest framing of the career impact question: the poles are fine, the middle is where the structure collapses.

---

## Key Themes

- **#concept** Simplicity scales with model intelligence — the two-tool Dzero architecture (bash + SQL) beats complex tool loops because smarter models need less structure, not more.
- **#concept** "Make it look like coding" — exploit models' training distribution by framing any problem as code generation. The Tailwind moment (inline styles over separate CSS files) is the canonical example.
- **#concept** Optimistic locking for shipping — invert approval gates: anyone can ship unless vetoed. Shifts the burden from "getting permission" to "someone stopping you."
- **#pattern** Solo exploration before team formation — one person finds PMF and produces a demo first. Teams are scaling mechanisms, not discovery mechanisms.
- **#pattern** The YAML semantic layer — Dzero's durable artifact isn't code, it's a plain-English description of every data column. The agent greps through it to understand the data. This is the [[Specifications as the Product]] pattern applied to data: the semantic description is the asset, the SQL is disposable.
- **#concept** Automation upgrades roles, doesn't eliminate them — 87% support automation meant support agents handle only hard problems. The career-ladder question (how do you train for the 13% without the 87%?) is left unanswered.
- **#tool** Dzero — internal Slack-based text-to-SQL: bash tool, Execute SQL tool, YAML semantic layer. ~50 lines of orchestration code.
- **#tool** V0 — Vercel's product agent, evolved from frontend tool to full-stack builder for non-engineers through four user-base pivots triggered by model capability leaps.
- **#person** Malte Ubl — CTO of Vercel, builder of [[json-render]], one of the most articulate voices on agent-native engineering.
- **#concept** "Free puppy" maintenance burden — creation cost → zero, maintenance cost unchanged. Every generated app creates a stewardship obligation. The equilibrium question: does cheaper creation mean more or fewer total engineers?

---

## Critical Analysis

**The strongest claim is the weakest documented.** "Building agents is actually extremely easy and you don't need to buy them" — said by the CTO of a company that sells an agent platform (V0). There's a tension here Ubl doesn't acknowledge. Vercel benefits whether you build or buy: if you build, you host on Vercel; if you buy, you buy V0. The "build, don't buy" advice costs Ubl nothing to give and makes him sound principled. The actual economics for a company without Vercel's infrastructure and talent density are likely different.

**The 50-line Dzero claim needs interrogation.** Ubl says the orchestration code is 50 lines. But the YAML semantic layer — the durable artifact that makes the whole thing work — is presumably maintained by humans who understand every Snowflake column well enough to describe it in plain English. That's the real cost center: not building the agent, but maintaining the semantic layer at scale. Every schema change requires a human to update a YAML description. As the data warehouse grows, so does the maintenance burden on the semantic layer. The 50-line agent is riding on a human-maintained knowledge base that might be the actual product.

**The V0 evolution story is a model-capability arbitrage play.** V0 didn't succeed because of product insight — it succeeded because Vercel was positioned to exploit each model leap before competitors could reposition. The Tailwind prompt hack (2023), the Sonnet 3.5 full-stack pivot, the current "tech-adjacent shadow IT" positioning — each is a fast-follow on a model release. The strategic question is whether this is sustainable or whether the window between "model ships" and "competitor catches up" shrinks to zero. Ubl's answer would probably be that Vercel's moat is in *running* software ("you throw shit over the fence and we're going to run it for you"), not in the agent that creates it.

**Optimistic locking is a trust-constrained pattern.** It works at Vercel because Vercel is a 750-person company with a strong engineering culture and a CTO who embodies the values. Does it scale to 5,000? To a company where engineering isn't the dominant culture? Ubl's answer is implicit in the headcount joke — he and the CEO joke about capping at 1,024, "notably below our hiring plans." Optimistic locking is a density-dependent pattern: it works when everyone knows everyone and trust is high. Beyond Dunbar's number, you need process.

**The "middle" engineer problem is the most important unresolved question and Ubl basically punts.** Senior ICs get minions, juniors are digital natives, the middle is "most interesting." That's not analysis — that's evasion. Mid-level engineers are the ones who do the work that senior ICs delegate and juniors can't handle yet. If the middle collapses, you lose the training ground for seniors and the support system for juniors. This isn't just an org chart question; it's a pipeline question for the entire industry.

**The 87% support automation number is a Rorschach test.** Ubl sees it as "agents have better jobs now." You could also see it as: 87% of support work was automatable, meaning those jobs were always going to be automated, and the remaining 13% is what the humans should have been doing all along. Or: 87% automation means 87% fewer entry points into the company for people without engineering degrees. Ubl's framing is optimistic but not wrong — the question is what "better job" means when the easy problems are gone and only the hair-on-fire problems remain, every single shift.

---

## Related Pages

- [[The Pragmatic Summit]] — the conference where this talk was delivered; full program and analysis
- [[json-render]] — Vercel's generative UI framework, also Malte Ubl's work
- [[Inside OpenAI's In-House Data Agent]] — parallel story: internal data agent built on company APIs, the metadata-as-semantic-layer problem
- [[What I learned building an opinionated and minimal coding agent]] — same minimalism thesis applied to coding agents: four tools, no MCP, competitive with Claude Code
- [[The solution might be cancelling my AI subscription (Willison)]] — the maintenance bottleneck Ubl describes as "free puppy"
- [[Specifications as the Product]] — the YAML semantic layer as durable artifact, generated SQL as disposable
- [[Webwright]] — "make it look like a coding task" applied to browser automation
- [[Tone LLM]] — the contract/adapter pattern: LLM fills a schema, code does the rest
- [[Thrifty (Tiered Delegation for Claude Code)]] — the tiered delegation pattern that Dzero's two-tool design inverts
- [[Minions — Stripe's One-Shot Coding Agents]] — another company's internal agent story
- [[The Founder's Playbook]] — the Idea→MVP stage before team scaling that Ubl advocates
- [[Running an AI-Native Engineering Org]] — Fiona Fung's field report, comparable organizational scale
- [[Uber — Agentic Engineering Shift]] — another large-scale agentic transformation
- [[Software Engineering at the Tipping Point]] — Adam Bender's 10× amplifier thesis, relevant to the career impact question
- [[The People Who Will Thrive in the AI Age]] — which engineers benefit, which don't
- [[Agent-Native Architectures (Every)]] — the design principles behind tools like V0
- [[The Pragmatic Summit]] — the conference context

---

*Source: [[summary/lessons-vercel-v0-d0-agent]]*
*Last updated: 2026-07-04*
