# An Honest Review of AI Programming

Mathieu Ropert's sober, experience-grounded assessment of LLMs for programming after three months of daily use: the tool's real value is in search and summarization, not code generation. His review is structured around four domains — LLMs as search engines (including the novel insight about internal company knowledge bases), hallucinations as a structural property (with concrete failure modes), code generation as consistently mediocre (with game dev as an acute training-data desert), and the dubious economics of an industry losing money on every customer. The piece is contrarian without being anti-AI: Ropert uses Claude daily but insists on "engineers design, agents implement" and always verifying against primary sources.

---

## Key Quotes

> "As long as you don't ask it to write code. Please don't ask it to write code."

Ropert's thesis, stated early and supported throughout. The LLM is a research assistant, not a programmer. This converges with [[Claude Is Not Your Architect]]'s "engineers design, agents implement" division of labor, but Ropert goes further: even *implementation* is a stretch. The tool writes mediocre code that costs more to review than it would to write.

> "I was able to find a bunch of answers that were written before my time. Because the same way 'AI Google Search' can run multiple queries in parallel and summarize the answer, Claude and friends can turn my natural language question into a bunch of search queries for likely synonyms until they hit something."

The most novel practical insight in the piece. Enterprise wikis and Slack histories are black holes of institutional knowledge — the information exists but can't be found because search is keyword-literal. An LLM's ability to map "UI" → "User Interface" and generate synonym queries bridges the gap. This is a use case that doesn't require the LLM to be *correct*, just *connected* — it finds the document, and the human verifies.

> "Claude was adamant that this was supported by past reports… until it turned out one of those was the one I was actually writing."

The self-reinforcing hallucination loop — a genuinely scary failure mode unique to LLMs with connectors into live systems. Your work-in-progress becomes its own supporting evidence. Ropert caught it because he checked sources; a less careful user would have created a circular citation chain. This pairs with the broader [[Hallucination and Truth]] problem but is more specific: it's not about invention, it's about the model treating *any* text in its context as authoritative.

> "Instead of 'just doing the thing', Claude had decided to apply some OOP design pattern that was uncalled for."

Ropert's concrete example: asked to move an `Update()` method into a manager class for a Unity optimization, Claude instead created a `GameUpdateable` base class with virtual `OnUpdate()`, introducing abstraction layers the explicit instruction avoided. This is the [[Claude Is Not Your Architect]] problem at the implementation level — the model's training-data bias toward OOP patterns overrides the engineer's design intent.

> "The last AAA game to be open sourced was Doom 3, a title that released in 2004, 22 years ago. It doesn't even have multithreading!"

The training-data poverty thesis applied to game development. LLMs trained on public GitHub repos see hobby projects, game jam entries, and tutorial demos. The actual production patterns — how AAA studios structure engines, handle memory, manage frame budgets — are proprietary and invisible. This extends [[AI Slop Starts with the Codebase Itself]] to the domain level: it's not just your codebase that's out of distribution, the entire *industry* is.

> "I keep this quote in mind a lot lately: 'It's interesting how AI is constantly providing false information and incorrect statements about my area of expertise. Fortunately, it's very useful and always right about topics I know very little about.'"

The @pikuma quote Ropert uses as a closing theme. This is the experience asymmetry that makes LLMs dangerous for learning: they're most convincing where you're least equipped to judge. Related to [[The Joy and Power of Understanding]]'s argument that you need force before the multiplier helps, and to [[Human-in-the-Loop is Tired]]'s diagnosis of supervision fatigue.

> "Management will not trust its engineers to spend their money on the tools they say they need, but then will tell everyone they have to use a shiny expensive toy they didn't request."

Ropert's institutional critique: the same managers who denied a €20/month profiler license are now mandating $20+/month Claude subscriptions. The decision isn't based on engineering ROI — it's top-down AI FOMO. This connects to [[AI Mania Is Eviscerating Global Decision-Making]]'s finding of AI investment at 0% success rate driven by executive prisoner's dilemmas.

---

## Key Themes

- **#concept LLM as enterprise search bridge** — The novel insight: LLMs fix broken enterprise search by generating synonym-rich queries that find information written in different terminology. Not a replacement for proper search indexing, but the pragmatic fix when you don't control the tooling. #pattern
- **#concept Self-reinforcing hallucination loops** — When LLMs with connectors cite their own outputs (or your WIP) as supporting evidence. A failure mode that doesn't exist in traditional search. Related to the telephone-game problem where chained LLM summaries drift from primary sources. #pattern
- **#concept Training-data deserts** — Some domains (game dev, custom engines, proprietary scripting languages) have effectively no training data from production systems. Models trained on hobby projects produce hobby-quality output. This is [[AI Slop Starts with the Codebase Itself]] scaled from codebase to industry. #concept
- **#concept OOP over-engineering bias** — LLMs default to adding abstraction layers even when explicitly told not to, because their training data skews toward OOP patterns. The model optimizes for what looks like "good code" in training distribution, not what the engineer asked for. #pattern
- **#pattern Verify everything, especially in unfamiliar domains** — Ropert's operating procedure: trust LLM output in areas you know well enough to catch errors, double-check everything in areas you don't. The @pikuma quote captures the asymmetry. #pattern
- **#concept The coding assistant framing** — LLMs as codebase-exploration tools, not code generators. Pull on threads, understand structure, find prior discussions. The "assistant" framing is more honest and more useful than the "developer replacement" framing. Stafford Beer's cybernetics provides the theoretical backing: you're limited by information-processing capacity, and an LLM can increase your throughput. #concept
- **#concept AI sustainability economics** — All major AI companies lose money. Marginal profit on tokens doesn't cover R&D and hardware. Prices are likely to rise, making current adoption driven by top-down mandates rather than ROI calculations. #concept

---

## Critical Analysis

**The enterprise search insight is the article's most original contribution and deserves more development.** Ropert identifies a real use case — LLMs as a bridge over broken enterprise search — that gets almost no attention in the broader AI-coding discourse, which is fixated on code generation and review. The observation that company wikis and Slack histories are "absurdly hard to access without an LLM" is true for most organizations, and the fix (synonym generation + parallel search) is genuinely something LLMs do well because it's summarization, not generation. This should be a product category.

**The game dev training-data poverty thesis is specific, falsifiable, and important.** It's also the strongest counterexample to the "AI will replace programmers" narrative. Ropert isn't arguing from principle — he's arguing from concrete domain knowledge: here's what production game code looks like, here's what's publicly available, and the gap between them is the entire ballgame. This extends [[AI Slop Starts with the Codebase Itself]] in a productive direction: some industries are *structurally* invisible to AI training, not just through proprietary patterns but because the artifacts themselves are never public.

**The OOP over-engineering example is a specific, falsifiable case study of [[Claude Is Not Your Architect]]'s thesis at the implementation level.** Holland argues AI shouldn't make architectural decisions; Ropert shows that even *micro-architectural* decisions (should I introduce a base class?) get over-engineered because the model defaults to patterns from its training distribution. The fix isn't a better prompt — it's that some tasks shouldn't be delegated at all.

**Ropert's sustainability argument is correct but under-specified.** "They're all losing money" is true but doesn't distinguish between investing for growth (Amazon in 2000) and structurally unsustainable unit economics. The comparison to management approving LLM subscriptions while denying profiler licenses is a sharper point: it's about who gets to decide what tools engineers use, and whether those decisions are grounded in engineering reality.

**The piece is strongest where it stays close to concrete experience.** The Doom 3 example, the PBR wild-goose-chase, the `GameUpdateable` over-engineering, the self-citing hallucination — these are specific, reproducible failure modes that generalizes beyond game dev. The weakest sections are the theoretical ones (Stafford Beer, the sustainability economics) which gesture at larger arguments without fully developing them. But as a field report from a skeptical practitioner, it's a valuable counterweight to both the AI-booster and AI-doomer extremes.

---

*Sources: [[raw/an-honest-review-of-ai-programming]], [[summary/an-honest-review-of-ai-programming]]*
*Last updated: 2026-08-06*
