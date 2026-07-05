# SLM Routing for Knowledge Workers

Mukul Singh argues the AI industry has it backwards: we optimize for developers who need frontier models, while the vast majority of AI users — knowledge workers in spreadsheets, email, and documents — don't need frontier capability at all. A nano-model classifier routing 70–85% of requests to cheap small models and 15–30% to frontier achieves 75–90% cost reduction with only 10 ELO points of quality loss. The #2 spot on the GDPval-AA leaderboard proves it.

---

## Key Quotes

> "Defaulting to the smartest model for every request is a waste strategy."

The article's single best line. It names the industry's unexamined assumption: that "best model" means "best model for every request." Singh's counter is empirical — a routed pair of models beats every single-model entry except the frontier itself, at a fraction of the cost.

> "Routing off-the-shelf models is step one. Step two is making small models better through targeted post-training."

This is where the piece gets interesting. The nano-router is a wedge — the real bet is on "hill-climbing": distillation + RL + domain adaptation producing small models that match frontier at 10× lower cost. Microsoft's MAI family is the proof-of-concept: MAI-Code-1-Flash at ~5B active parameters beating Claude Haiku 4.5 on all coding benchmarks.

> "The right model should be automatic rather than bigger."

The closing thesis. Not "make the biggest model cheaper" but "make model selection invisible." This is a UX argument dressed as infrastructure: the knowledge worker shouldn't know or care which model handled their spreadsheet formula.

> "For developers, the difficulty distribution is flatter, the action space is unbounded, and the cost of errors compounds."

The honest carve-out. Singh isn't arguing routing works everywhere — he explicitly acknowledges the developer use case is different. This is what keeps the piece from being cargo-cult: it's a *segmented* argument, not a universal one.

## Key Themes

#tool #concept #pattern #comparison

**Model Routing.** A nano-model classifier decides per-request whether to use a cheap small model or an expensive frontier model. Overhead: <$0.01 per request. The classifier locks the model for the entire session to preserve prompt caches — a practical detail that separates real infrastructure from thought experiments.

**Hill-Climbing.** Microsoft's term for the methodology of making small models better through distillation, reinforcement learning, and domain adaptation. Distinct from just routing off-the-shelf models — the goal is to systematically close the quality gap between small and frontier models on specific domains.

**Knowledge Worker vs. Developer.** A segmentation the industry rarely makes. Knowledge workers have bounded action spaces (the operations available in Excel are finite), steep difficulty distributions (most requests are formatting or formula-lookup, not novel reasoning), and latency sensitivity (they won't wait 30 seconds for a response). Developers have the opposite profile.

**GDPVal.** A benchmark measuring AI performance on knowledge-worker tasks (spreadsheets, documents, email). The routed pair of GPT-5.5 + GPT-5.4 Mini hit #2 on the leaderboard, losing only 10 ELO points to pure GPT-5.5 while costing >10× less. The benchmark itself validates the routing thesis by showing the difficulty distribution IS steep.

## Critical Analysis

**The strongest claim is also the most fragile.** The 10 ELO point gap between routed and pure frontier is compelling, but it's one benchmark on one model pair. We don't know if the routing classifier generalizes across model families (would the same nano-model route well between Claude Opus and Claude Haiku? Between Gemini and Gemini Flash?), or if the GDPVal difficulty distribution is representative of real knowledge-worker workloads. The article presents routing as a general solution backed by a single data point.

**The developer carve-out is theoretically right but practically suspicious.** Singh says routing doesn't apply to developers because their difficulty distribution is flatter. But the Thrifty plugin showed Sonnet/Haiku routing achieved ~64% cost savings at equal quality for coding tasks. The Advisor Strategy from Anthropic formalizes the same pattern. Singh's clean segmentation between "knowledge workers" and "developers" may be a useful rhetorical device rather than an empirical finding — in practice, coding tasks also have a steep difficulty distribution, and coding agents are already tiered.

**The MAI section reads as Microsoft advocacy grafted onto an independent argument.** The first half of the piece is a routing argument anyone could make. The second half is a product tour of Microsoft's MAI models. The "trained from scratch on clean, licensed data without third-party distillation" line is a pointed jab at OpenAI — and it's Microsoft making it, which is worth noting given the Microsoft-OpenAI relationship. The piece would be stronger if it acknowledged the vendor interest or separated the general routing argument from the MAI case study more clearly.

**"Default to routing, not to frontier" as product philosophy, not just infrastructure.** If Singh is right, the implication isn't just "add a router to your API calls." It's that every AI product surface — Copilot, Claude, Gemini, ChatGPT — should route by default and only surface the frontier model when the task demands it. That's a product decision, not an engineering one, and it conflicts with the "one model to rule them all" marketing that currently drives adoption. The quietest but most important implication: routing means users stop knowing which model they're using, which means the model brand becomes infrastructure, not product.

**What's missing:** No discussion of failure modes. What happens when the nano-classifier routes a hard task to the small model? What's the error profile — silent wrong answers, or obviously inadequate responses? For knowledge workers doing financial analysis, a silent error from a misrouted request could be more expensive than the entire cost savings of the routing system. The piece needs a section on routing safety: confidence thresholds, escalation paths, and user-visible signals when the system is operating at the edge of its competence.

**The real insight:** Singh's argument is really about *task characterization*, not model capabilities. The key move is defining knowledge-worker tasks by their structural properties (bounded action spaces, steep difficulty distributions) and then matching infrastructure to those properties. This is the same move that [[Smart Models Dumb Pipes]] makes — let the model own judgment, but make the infrastructure own the decision of *which* model. The router is the smartest dumb pipe.

---

*Sources: [[summary/slm-routing-knowledge-workers]]*
*Last updated: 2026-07-05*
