# HN RIP Low-Code 2014-2025

A 157-comment Hacker News thread that uses Zack Liscio's "RIP Low-Code 2014-2025" obituary as a launching pad for a deeper argument about what low-code actually was, why it's being displaced, and what survives the AI collision. The crowd is highly experienced — founders of low-code companies, engineers who've migrated off platforms, and skeptics who've heard this death knell before. The consensus is that Liscio is partly right about the short-term disruption but wrong about the long-term death: low-code isn't dying, it's being redefined as the abstraction layer AI agents operate on.

---

## Key Quotes

> "LLMs work much better with solid abstractions; they are not great at coding the whole thing from scratch. More code means more tokens, more errors, less maintainable codebase. Low code has become *more* important with LLMs because the less code you have per feature, the more stable things are."

**socketcluster** pushes back hardest against the article. This is the central tension in the entire thread: does AI make low-code redundant (by generating code so fast) or more essential (by needing smaller, cleaner targets to generate into)? The thread doesn't resolve this — it surfaces it as the live question.

> "LLMs can assist you to write a shitload of useless bloatware, or can assist you to take something existing and complicated, and create something minimal."

**antirez** with the thread's sharpest distinction. This connects directly to [[Simplicity in the Age of AI-Assisted]]'s argument that LLMs are better demolition tools than construction tools. The question isn't how much code AI can generate — it's what direction it pushes you.

> "Fuck all this pointless noise… Just someone give me MS Access for the web with an SSO module."

**dgxyz** with the thread's most visceral comment, which sparked a long sub-thread of Access nostalgia. People who'd built real ERP systems in Access defended it against those pointing out corruption and scaling issues. The longing is for a tool where domain experts can build real things without learning npm, bash, and AWS.

> "Writing code is the 'easy' part and kind of always has been. Shipping code, monitoring code, fixing code, and upgrading code is the hard part."

**oxidant** captures the near-universal pushback against Liscio's "cost of shipping code approaches zero" premise. The thread is unanimous on this: code generation cost is a rounding error in total system cost.

> "What do you think an LLM is if not no/low-code? You type in natural language and it gives you an app."

**_pdp_** reframes the entire debate. If low-code means "builder doesn't write code," then LLMs are the ultimate low-code platform. The fact that they generate "high code" behind the scenes is an implementation detail — end users don't care. This is the argument that most threatens the "low-code is dead" thesis: low-code won, it just looks different now.

> "AI produces code that only AI maintains."

**phartenfeller** distills the maintenance anxiety. Low-code vendors ship stable, composable patterns with tested upgrade paths. AI generates one-off code with no upgrade story. This is the durable advantage low-code platforms have — and the one most likely to survive AI disruption.

> "Low code wont die. But selling low code might."

**hahahahhaah** with the thread's most quotable one-liner. The category survives; the business model of proprietary platform lock-in may not.

## Key Themes

- **#concept Low-code as AI abstraction layer** — Multiple commenters argue low-code's future is as the constraint surface for AI agents. Give agents smaller, simpler primitives and they perform better. This inverts Liscio's thesis: rather than replacing low-code, AI makes it more valuable as a target.
- **#pattern Build vs. buy inversion** — The thread validates Liscio's core experience (Retool → AI-built replacements) but complicates it. The build-vs-buy calculus now depends on whether you have AI-competent developers and what "buy" means when platforms are adding AI themselves.
- **#concept Maintenance as the real cost** — The thread's strongest consensus. Every commenter with production experience agrees: generating code is cheap, maintaining it is expensive. See [[Write Only Code]] for the Slop Radius framing, [[Cognitive Debt]] for what accumulates when AI code outpaces comprehension.
- **#tool MS Access / VB6 as the lost golden age** — A surprising nostalgia thread. Access had real problems (corruption, scaling) but solved real problems (domain experts building useful tools without professional developers). The thread asks: why can't we do better than Access after 25 years? The answer is partly that Access's simplicity was tied to a single-machine, single-user model that doesn't generalize to the web.
- **#concept MCP and GraphQL as the new integration surface** — Davidpolberger (Calcapp) is betting on MCP-first architecture. Others argue GraphQL introspection makes it better for agent discovery. A sub-debate on REST vs GraphQL vs Swagger reveals how unsettled the agent-to-system integration layer still is. See [[Building Agents for Production Systems with MCP]].
- **#person Citizen developer future** — Will non-technical users embrace vibe-coding or prefer visual tools they can reason about? The thread splits: some see AI making everyone a developer, others see visual low-code tools as more accessible because they show the system's structure.
- **#concept Code volume as liability** — socketcluster's core argument: more code means more tokens, more errors, more maintenance. This inverts the AI optimism — if AI makes code free, it also makes code bloat free. The scarce resource isn't code generation, it's code comprehension.

## Critical Analysis

**The thread is more valuable than the article it discusses.** Liscio's piece is a startup post-mortem with a thesis bolted on — specific, honest, but narrow. The HN discussion is a 157-person focus group of practitioners who've been building in this space for years. The signal-to-noise is unusually high for HN.

**The "low-code isn't dying, it's becoming the AI abstraction layer" argument is the most compelling synthesis.** If AI agents work better with constrained primitives (see [[Honey I Shrunk the Coding Agent]]), then low-code platforms aren't competing with AI — they're providing the constraint surface AI needs to be reliable. The platforms that survive will be the ones that understand this and reposition as AI-native development environments, not alternatives to code.

**The MS Access nostalgia is diagnostic, not prescriptive.** Nobody actually wants MS Access for the web. They want what Access represented: a tool where domain expertise was sufficient to build useful software. The thread reveals that 25 years of web development have made that *harder*, not easier — you now need to understand deployment, auth, APIs, databases, and infrastructure. Low-code promised to fix this but delivered lock-in instead. AI might actually deliver what low-code only promised, but the thread is rightly skeptical.

**The maintenance argument is the thread's most durable insight.** Code generation is a solved problem (or close enough). Code maintenance is not. The thread's strongest voices — socketcluster, phartenfeller, oxidant — all converge here. This is the argument that will age best: whatever wins in the low-code vs. AI battle, the cost structure of software shifts from creation to stewardship. This aligns with [[The Future of Software Engineering is SRE]] and [[Write Only Code]].

**The thread misses the security dimension almost entirely.** For 157 comments on replacing platforms with AI-generated code, almost nobody mentions security, compliance, or audit. The same blind spot [[AI Coding Tools Create More Bugs Than They Fix]] documents — AI builders don't know what they don't know about security. The closest the thread gets is zerktens's point about organizational compliance, but it's a brief mention in a sub-thread.

**Liscio's "cost of shipping code approaches zero" is the thread's punching bag for good reason.** Nearly every experienced commenter calls it out. The phrase conflates "writing code" with "shipping code" — two activities with wildly different cost profiles. But the thread's correction, while accurate, doesn't fully engage with Liscio's actual experience: his team did ship, and it was faster without Retool. The gap between "shipping code is expensive" (true) and "AI made it cheap enough to beat Retool" (also true, per Liscio's data) is where the interesting argument lives.

---

*Sources: [[summary/hn-rip-low-code-2014-2025]], [RIP Low-Code 2014-2025](https://www.zackliscio.com/posts/rip-low-code-2014-2025/)*
*Last updated: 2026-05-15*
