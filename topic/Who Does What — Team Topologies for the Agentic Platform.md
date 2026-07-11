# Who Does What — Team Topologies for the Agentic Platform

Olivier Wulveryck extends Team Topologies (Skelton & Pais) to the agentic era, reframing the core problem: cognitive load doesn't vanish with AI, it transforms into **anticipation burden** — everything a human must foresee before launching an agent, compressed into ever-shorter decision windows as agents produce continuously. The answer is an *agentic platform* that absorbs systemic complexity, letting business teams drive production through agents while developers shift from building applications to building the platform itself. The article is both a proposal and an open debate — Wulveryck surfaces HN's counterarguments directly in the text rather than dismissing them.

## Key Quotes

> "Cognitive load does not disappear with AI — it transforms. It first becomes an anticipation burden"

The sharpest reframing in the article. The old problem was "too much complexity for one team to hold." The new problem is "too many decisions arriving too fast for one human to regulate." Throughput, not quantity.

> "the developer who carries the anticipation burden: they know which questions the agent will not ask"

This is the quiet insight that justifies the entire platform investment. Agents are pathologically incurious — they'll barrel ahead with wrong assumptions rather than ask clarifying questions. The platform must encode the questions the agent won't think to ask.

> "Enabling disappears because it succeeds, not because it fails"

The most elegant team design in the model. Enabling teams are structurally temporary — they exist to make themselves obsolete. This is the opposite of most organizational functions, which grow to fill available budget.

> "Ease of production must be matched by ease of oversight"

The shadow IT warning, compressed to an aphorism. When any business team can deploy, governance can't be a separate process — it must be baked into the deployment itself. Systemic visibility is the only thing that scales.

> "Autonomy is not silence: even at maturity, product teams remain the sensors that feed graduation"

The graduation mechanism (rule of three: when three teams need the same guardrail, it gets systemized) is only as good as its sensors. If stream teams go quiet, the platform fossilizes.

## Key Themes

- **#pattern** — Team Topologies adapted for agentic engineering: four team types with explicit interaction modes
- **#concept** — Anticipation burden: the cognitive load shift from "holding complexity" to "regulating decision throughput"
- **#concept** — Graduation: the rule-of-three mechanism for promoting product-specific guardrails to systemic platform features
- **#concept** — Systemic vs. dynamic context: the clean split between what the platform provides and what product teams own
- **#person** — Olivier Wulveryck: French engineer thinking out loud about organizational design for the agentic era

## Critical Analysis

**What Wulveryck gets right:** The anticipation burden diagnosis is the real contribution here. Everyone talks about cognitive load; almost nobody talks about cognitive *throughput*. Agents don't wait for you to catch up — they keep producing, and the bottleneck shifts from "can you understand this?" to "can you decide fast enough?" The platform-as-absorber model is the right architectural answer to that problem.

**What's underspecified:** The article is essentially an org chart in search of a technical architecture. What *is* the agentic platform? Is it a set of MCP servers? A coding agent harness? A deployment pipeline with guardrails? The team topology is the easy part; the hard part is building a platform that genuinely absorbs anticipation burden rather than adding another layer of indirection. Wulveryck gestures at "systemic context" and "systemic guardrails" but never gets concrete about what they're made of.

**The self-awareness is notable but limiting:** Including HN criticism in the article is intellectually honest, but it also makes the piece read like a design document with the review comments left inline. The Agentic Waterfall concern is real — if you're not careful, you've just reinvented the PR/QA/deploy pipeline with AI agents playing each role, complete with the same handoff delays. Wulveryck acknowledges this but doesn't resolve it.

**Compared to Johnson's [[The Case Against Building Your Own Agent Platform]]:** Johnson argues building the platform is a trap for most organizations. Wulveryck is describing the org chart for the team that would walk into that trap. The tension is productive: Johnson says "buy the platform, build the agents"; Wulveryck says "here's how to organize if you're building the platform." They're not contradictory — they address different audiences at different scales.

**The graduation mechanism is underrated:** The rule of three is a genuinely good governance pattern — it prevents premature abstraction (don't platformize something only one team needs) while ensuring discoveries propagate. But it only works if the platform team has the capacity to absorb new guardrails. At scale, this becomes the platform product owner bottleneck Wulveryck himself warns about.

**Why this matters now:** As [[Running an AI-Native Engineering Org]] documents, the bottleneck in AI-native engineering has already shifted from coding to verification. Wulveryck's model pushes further: the next bottleneck is *decision-making velocity*, and the only way to increase it is to move decisions into the platform where they execute automatically rather than requiring human attention.

## Related Pages

- [[The Case Against Building Your Own Agent Platform]] — Johnson's build-vs-buy triage; the platform Wulveryck describes is exactly what Johnson warns against building yourself
- [[Platform Engineering End-to-End]] — Cavallin's field guide covers the same organizational ground without the AI lens
- [[Running an AI-Native Engineering Org]] — Fung's field report on what actually changes when coding stops being the bottleneck
- [[Guardrails and Feedback Loops]] — The technical implementation of what Wulveryck calls systemic guardrails
- [[Agent Orchestration]] — Hub page for multi-agent coordination patterns
- [[Loop Engineering]] — Osmani's meta-skill of designing systems that prompt agents; the individual contributor's version of platform thinking
- [[The Agentic Product Standard v2.0]] — The closest thing to a specification for the platform Wulveryck's teams would build
- [[Agentic Software Engineering (Hassan)]] — Hassan's comprehensive treatment of the same transformation at the discipline level
- [[The Persistent Gravity of Cross Platform]] — Pike's argument that coordination costs drive teams toward cross-platform tools is a case study in the anticipation burden Wulveryck describes; the 2026 addendum linking agentic coding to verification-as-bottleneck maps directly to the platform's role as a decision-velocity multiplier

---

*Source: [Who Does What? Team Topologies for the Agentic Platform](https://blog.owulveryck.info/2026/06/22/who-does-what-team-topologies-for-the-agentic-platform.html) by Olivier Wulveryck, 2026-06-22. Fetched 2026-06-24.*
