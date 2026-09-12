# AI Value Chain

A systematic analysis of where durable, defensible value sits across the AI stack — from foundation models through deployment services to enterprise applications — and the quiet war between frontier labs and their own customers over who captures it.

---

## The Core Argument

Lhl (devstack) lays out the case that frontier AI labs — OpenAI, Anthropic, Google, Microsoft — are engaged in a structural value extraction play against their enterprise customers, not primarily through training on customer data (the obvious fear) but through three subtler mechanisms: **forward-deployed engineers** harvesting reusable artifacts from deployments, **platform control** used to enter customer markets, and **engagement byproducts** (tasks, rubrics, expert decisions) that become proprietary training assets worth low-to-mid seven figures per engagement.

The economic logic is straightforward: foundation model access is collapsing toward zero margin (GLM 5.2 at ~$1.40/M tokens vs. Opus 4.8 at $5/M), training costs tens of billions, and the only durable moat is the **data flywheel** at the deployment surface. So labs move up the stack — services (DeployCo), first-party apps (Claude Code, Cowork), and the "discovery layer" (Claude Science). Every move toward the customer is also a move toward the training signal that nobody else can get.

## The Three Questions

Lhl separates what's usually conflated into three distinct questions, each with different answers:

| Question | Verdict | Why |
|----------|---------|-----|
| Are labs secretly training on enterprise data? | **Weak** (today) | All four major providers contractually say no. Azure adds architectural separation. But defaults change, and OpenAI already did it once (InstructGPT before March 2023). |
| Do FDE engagements produce reusable artifacts? | **Strong** | Labs describe it themselves. A third-party market (Surge, Mercor, Mechanize) prices these goods. Expert-validated tasks cost $200–$2,000 each. |
| Can labs use platform control to compete with customers? | **Strong** | Windsurf's Claude access cut during OpenAI acquisition talks. Cowork plugins triggering Thomson Reuters/RELX selloffs. Claude Code competing with Cursor. |

The training question gets the headlines but the byproducts and competition questions are where the action actually is. This is the most useful frame in the piece.

## Key Evidence

### Windsurf
Coding-assistant startup in acquisition talks with OpenAI. Anthropic cut their Claude API access. The implied threat: platform access as a competitive weapon, and one that doesn't require model training — just API key revocation.

### Bridgewater
The hedge fund tested frontier models on internal document tasks and got ~50% accuracy. Then they fine-tuned an open Chinese model and hit 84.7% at ~1/14th the cost. The result suggests frontier models saturate on **tacit organizational judgment** — the kind of knowledge that can't be prompted into a model but can be fine-tuned into one. This is the most important empirical finding in the piece and has implications nobody seems to have fully processed.

### Coinbase
Single internal LLM gateway. 91% of engineers never hit usage caps. Spend down 50% while token usage grew. Projected 80% of workloads on 99%-cheaper models within 12–18 months. The gateway as demand-aggregation surface is the enterprise's counter-weapon.

### Claude Code Steganography
Anthropic's client silently varied Unicode characters in a system prompt to encode endpoint detection of Chinese AI labs using the API. Lhl treats this as a trust-break on the level of the training question: if the client is silently encoding tracking data into prompts, what else is it encoding?

## Four Countermeasures

Lhl identifies four enterprise countermeasures with public evidence:

1. **Fine-tune your way out**: Bridgewater proved open models beat frontier on domain judgment. The moat isn't the model — it's the tacit knowledge embedded in your organization's decisions.

2. **Own the gateway**: Coinbase's single routing layer enforces data policy in code, keeps multi-provider portability, and returns demand-visibility to the enterprise. Without the gateway, you're feeding pricing and usage intelligence to every provider equally.

3. **Sovereignty as bargaining chip**: France's DGSI replacing Palantir with ChapsVision. Germany's BfV doing likewise. Spain blocking new Palantir contracts. UK reviewing the NHS deal. European governments are building the market leverage that individual enterprises can't.

4. **The gateway pattern** (elevated to its own countermeasure): A single internal LLM routing layer is the enterprise's demand-aggregation surface. Whoever owns it owns the data flywheel. This is the architectural answer to the platform's structural advantage.

## The Stratified Truce

Lhl's equilibrium model: usage and learning both stratify by difficulty and ownership. Frontier models keep the hardest tier. Open/custom models absorb high-volume work. Services are the contested middle.

Four things break the truce:
- A closed lab solves **continual learning** (every deployment log becomes training signal)
- Chinese open weights become **unusable for regulated Western enterprises**
- An **AI financial correction** resets investment assumptions
- Frontier models prove they can **originate lucrative discoveries** (Claude Science, drug discovery)

Of these, continual learning is the existential one. If the firewall between deployment and training disappears, the enterprise playbook needs to be rewritten from scratch.

## Critical Analysis

**What this piece gets right**: Separating training, byproducts, and competition into distinct questions is genuinely clarifying. Most writing on this topic collapses them into "are they training on our data?" which is defensible today, and misses the real action. The Bridgewater result is the most important AI strategy finding of 2026 so far and deserves its own deep-dive.

**What it underweights**: The enterprise playbook section is solid but assumes a level of procurement sophistication that most companies don't have. The gateway pattern requires engineering capacity that only Coinbase-tier organizations possess. And the sovereignty countermeasure is real for governments but doesn't help SaaS companies competing against Cowork plugins.

**What's missing**: The piece treats "labs" as a monolith but the strategies differ. Anthropic's Mythos retention requirements and Claude Code steganography are qualitatively different from OpenAI's DeployCo play. Google's strategy is less examined. And Microsoft's position as both hyperscaler and model provider creates conflicts the piece touches but doesn't fully map.

**The most uncomfortable implication**: If the byproducts thesis is right — and the evidence says it is — then every enterprise FDE engagement is a value transfer from customer to lab that isn't priced into the contract. The lab gets reusable assets. The customer gets a deployment. Only one of those appreciates.

**Bottom line**: Required reading for anyone buying AI at the enterprise level, and a useful structural map for everyone else. The Bridgewater result alone is worth the time.

## Key Themes

#concept [[AI Value Chain]] #concept [[Data Flywheel]] #pattern [[Forward Deployment]] #pattern [[Gateway Pattern]] #tool [[Open Weights Models]] #comparison [[Open vs Closed Models]]

---
*Sources: [[summary/ai-value-chain]]*
*Last updated: 2026-07-05*
*Cross-references: [[How Far Behind Are Open Models]], [[GLM-5.2 Is the Step Change for Open Agents]], [[Open models lag state-of-the-art closed models by 4 months]], [[Notes from the AI Now Summit by Mistral]], [[Muse Spark and the Rough Edges Admission]], [[AI Cybersecurity After Mythos — The Jagged Frontier]], [[Who Owns the Code Claude Wrote]], [[The Flat Curve Society]], [[2026 Global Intelligence Crisis]], [[Inference Cost Napkin Math]], [[How AI Labs Are Solving the Power Crisis]], [[The Education of the Broligarchy]], [[Why Agents Matter More Than Other AI]], [[Introducing Claude Tag]], [[The Founder's Playbook]], [[2025 in LLMs]], [[AI Livestream Factories]], [[Cohere North Mini Code]]*
