# MCP Is Dead; Long Live MCP

Charles Chen's sharp rebuttal to the influencer-driven pendulum swing from MCP-mania to CLI-everything. The binary framing is wrong: local MCP over stdio is often unnecessary complexity, but **MCP over streamable HTTP is transformative for organizations**. The overlooked distinction — between tools, prompts, and resources — makes MCP a delivery mechanism for org-wide engineering standards, not just a tool protocol. This is the most nuanced MCP piece in the wiki, and the only one that takes the CLI-first position seriously while dismantling its enterprise applicability.

---

## Key Quotes

> "it's just an API; why do I need a wrapper around that when I can just call the API directly."

Chen's initial reaction to MCP vendors — the correct default for solo developers. If you can call an API directly, do that. The wrapper argument only makes sense when you need what MCP over HTTP provides: centralized auth, telemetry, dynamic content delivery, and org-wide distribution.

> "congrats on buying into the current AI-influencer FOMO hype cycle; see you in 6 months."

The article's bite. Chen is contemptuous of the influencer churn — first MCP, now CLI, whatever's next — not because the technologies are wrong, but because the discourse has abandoned engineering tradeoff analysis for engagement farming. He lumps Garry Tan and Andrew Ng in with "influencers peddling Ivermectin as a cure-all or anti-vax conspiracies; it's simple ignorance." Harsh, but the point isn't that Tan and Ng are wrong about CLIs — it's that their individual success stories don't generalize to teams managing security, compliance, and maintenance.

> "MCP is currently the right tool for orgs and enterprises."

The thesis, stripped to one sentence. Not the right tool for everyone, and certainly not for solo developers who can `curl` an API. But for organizations that need consistent auth, telemetry, dynamic content, and standardized delivery of skills and docs across teams — MCP over HTTP is the mechanism.

> "If you want to deliver standard planning prompts (skills) and standard docs across all repositories... consistently and always up-to-date with server telemetry to measure adherence and usage across an org, then MCP prompts and resources IS that delivery mechanism."

Chen's most original contribution. The conversation around MCP has been dominated by tools — but prompts (effectively "server-delivered SKILL.md") and resources (effectively "server-delivered /docs") are the real enterprise value proposition. This is server-side content with dynamic injection, automatic updates, and org-wide telemetry for measuring adoption. [[Control Plane MCP Server]] executes exactly this pattern with their virtual resources and curated prompts.

> "what works individually in a homogeneous environment likely will not hold true for teams and orgs."

The closing warning. YC partners and AI researchers shipping solo projects from a consistent MacBook setup are not a representative sample. The engineering challenges that matter at scale — auth rotation, telemetry, compliance, onboarding, knowledge distribution — don't exist in that environment, so their tool preferences don't address them.

---

## Key Themes

#concept #tool #pattern #mcp #agent-architecture

### The local/remote MCP split is the missing variable

The entire CLI-vs-MCP debate collapses into incoherence without distinguishing stdio MCP from HTTP MCP. Chen grants the CLI-first position for local use — stdio MCP adds boilerplate for no benefit. But remote MCP over streamable HTTP solves problems CLIs can't touch: centralized auth, telemetry, dynamic content delivery, and single-point-of-update for org-wide prompts and resources. Most of the MCP critique is really a stdio critique, and Chen is right that the critics rarely acknowledge the HTTP case.

This connects directly to [[What I learned building an opinionated and minimal coding agent]]'s finding that minimal CLI tools outperform MCP for solo work, and [[Designing Agentic Loops]]'s argument that shell commands beat MCP. Both are correct for the stdio case — and both are silent on the HTTP case. Chen fills that gap.

### Influencers are optimizing for engagement, not engineering tradeoffs

Chen's most provocative claim: the AI influencer ecosystem has structural incentives to oscillate between extreme positions (MCP is everything! MCP is dead!) because hot takes drive engagement. The real engineering question — "in what contexts does MCP make sense, and for whom?" — is too boring for the algorithm. The result is that teams building serious production systems have to tune out the same voices that shaped their initial architecture decisions.

This is the same dynamic that [[Vibe Coding and the Maker Movement]] identifies as "evaluative anesthesia" — the dopamine of participation eclipsing the ability to judge. Chen is arguing for evaluative sobriety.

### MCP prompts and resources are the sleeper feature

The tool definition layer got all the attention, but Chen argues prompts and resources are MCP's real enterprise value:
- **Prompts** = server-delivered SKILL.md with dynamic content injection (pricing, system status, context)
- **Resources** = server-delivered /docs that stay current without manual sync

This is a genuine insight. The wiki's [[Building Agents for Production Systems with MCP]] page notes the Skills + MCP pairing but doesn't go as far as Chen in positioning prompts/resources as *the* reason to adopt MCP. [[Control Plane MCP Server]] is the reference implementation: virtual resources embedding documentation, curated prompts as skill directory, all served from a central server with telemetry.

### From vibe-coding to agentic engineering

Chen's call to action: organizations need to move beyond "cowboy, vibe-coding culture" toward "organizationally aligned agentic engineering practices." MCP over HTTP is the infrastructure for that transition — not because it's a better tool protocol, but because it centralizes the things organizations need to manage: auth, telemetry, content distribution, standards enforcement.

This aligns with [[Compound Engineering]]'s argument for systems over manual review, [[Guardrails and Feedback Loops]]'s position that linters beat prompts, and [[Agent Coding Workflow]]'s maturity spectrum from vibes to compound engineering. Chen is describing the organizational infrastructure that makes Level 4-5 possible at scale.

---

## Critical Analysis

**This is the best MCP analysis in the wiki.** Not because Chen is pro-MCP (he's not — he's pro-MCP-in-context), but because he grants the strongest version of the anti-MCP argument before explaining why it only applies to one deployment model. Most MCP discourse is people talking past each other about different things. Chen names the thing: stdio vs. HTTP, tools vs. prompts/resources, solo vs. org.

**The influencer critique is the weakest part.** Chen is right about the structural incentives, but the Ivermectin comparison is rhetorically overheated in a way that undermines the measured engineering argument. Garry Tan and Andrew Ng may be wrong about MCP's enterprise applicability, but they're not selling snake oil — they're reporting their experience. The problem isn't that they're wrong about their own workflows; it's that their audience treats individual experience as universal truth. That's a media literacy problem, not a Tan/Ng problem.

**The Amazon citation cuts both ways.** Chen cites Amazon requiring senior engineer sign-off on AI-assisted changes as evidence that organizations need MCP-based discipline. But the Amazon example actually shows that organizations impose process constraints *regardless of protocol choice* — if Amazon's problem was quality, adding MCP doesn't fix it. The fix was human review. MCP might reduce the *surface area* of quality problems (consistent prompts, standard docs, centralized auth) but it doesn't replace the need for human judgment at the review point.

**What's missing:** Chen doesn't address the MCP adoption tax. Setting up an HTTP MCP server with OAuth, OpenTelemetry, dynamic content generation, and subscription management is non-trivial infrastructure. The "just point to an HTTP endpoint" claim papers over significant operational complexity. For organizations that already have service infrastructure, this is manageable — for teams that don't, it's a chicken-and-egg problem. [[How Intercom Uses Claude Code]] is the enterprise case study (13 plugins, 100+ skills, OpenTelemetry), but Intercom had the ops team to build it. Most orgs don't.

**The real contribution:** Chen has identified a gap in the discourse that this wiki partly reproduces. We have pages praising MCP ([[Building Agents for Production Systems with MCP]], [[Agency]]) and pages arguing for CLIs ([[Designing Agentic Loops]], [[10 Principles for Agent-Native CLIs]], [[What I learned building an opinionated and minimal coding agent]], [[surf-cli]]). What we didn't have was a page that maps the boundary between them. This is that page.

---

*Sources: [[raw/mcp-is-dead-long-live-mcp]]*
*Last updated: 2026-05-15*
