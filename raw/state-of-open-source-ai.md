---
url: https://stateofopensource.ai/
title: The State of Open Source AI — V1.0
author: Mozilla (Raffi Krikorian, CTO)
date_fetched: 2026-07-18
date_published: 2026-07
---

# The State of Open Source AI — V1.0 · July 2026

This is a Mozilla-published report (July 2026) assessing the open-source AI ecosystem. No sponsor information is disclosed beyond Mozilla itself.

---

## Opening Letter — Raffi Krikorian, CTO of Mozilla

Krikorian opens with vignettes: a Māori broadcaster training speech models for te reo under a data-sovereign license; PwC fine-tuning an open model on finance language running for hundreds of clients on its own hardware with no per-token meter; researchers in Lausanne building an open medical model with the Red Cross for clinical trials in Switzerland and Tanzania; East African farmers diagnosing cassava disease offline on-device; a Swiss public consortium releasing a national model's weights, data, and training code. His central thesis: "They own it — that is the whole idea."

He draws a parallel to Mozilla's origin story — one company trying to own the web's front door, an open community rising up in response. He argues the same play is being run again. The path forward, he says, is "competition and interoperability," a world of many models with standard ways to connect them and the ability to leave any vendor. He frames the rest of the document as a map of where open AI is winning and where it is exposed.

---

## Section 1: The Current State of Open-Source AI

**Headline statistic: Capability gap to top closed models at 0% (parity on coding, behind on reasoning). GPT-4-class inference cost fell 50× in 36 months: from $20 to $0.40 per 1M tokens.**

### Capability Gap

The gap between open and closed models on Chatbot Arena over 24 months:
- Started at 8.04%
- Collapsed to 0.5% by August 2024
- DeepSeek-R1 briefly matched the top US model in February 2025
- Reopened to 3.3% by March 2026 as closed reasoning models pulled ahead

The 3.3% is an average over a "jagged frontier": open is at or near parity on coding, instruction-following, and general knowledge. The gap concentrates in reasoning, long-context retrieval, and agentic tasks. Source: Chatbot Arena, Jan 2024–Mar 2026.

### Inference Cost Collapse

GPT-4-equivalent pricing dropped from ~$20 to ~$0.40 per 1M tokens — described as faster than dotcom-era bandwidth or PC-compute price curves. Sources cited: Stanford HAI AI Index 2025 (280× GPT-3.5-class drop over 18 months); Epoch AI (9–900× annual decay); Nov 2025 MIT study (5–10×/yr at the frontier, hardware-adjusted).

### Token Routing (OpenRouter)

Open-weight models' share of tokens routed on OpenRouter grew from negligible to roughly a third by late 2025 and to a **majority by mid-2026**. Source: OpenRouter 100T-token study (Nov 2024–Nov 2025) and live leaderboard; intermediate points interpolated. The report notes that by request count, closed US providers still lead — the open-model lead is a token-volume lead concentrated in coding and agentic workloads.

**The five highest-volume models on OpenRouter's live leaderboard are all open weights.** Anthropic's closed Claude models are the next US-built entrants.

By mid-2026, the top nine models route roughly **18T weekly tokens** for Chinese-built models vs. ~5.5T for US-built ones — more than a 3:1 ratio (FT analysis). Where developers route by cost, they route to open weights.

### Adoption vs. Production Gap

Based on the **Mozilla / SlashData 2026 developer survey**:

- **79%** of developers adding AI functionality use open models
- **71%** use closed models
- The two are largely complementary: **29% use open only, 50% use both, 21% use closed only**

**Production rate** — only **51% of open-model teams reach production** vs. **63% for closed**. The report attributes this gap to operational tooling and trust, not capability.

**Production rate by company size**: Closed climbs from 54% to 73% with scale; open barely moves from 53% to 57%. Enterprises can buy their way through closed deployment; open deployment "waits on tooling nobody has finished."

### Why Teams Churn From Open Models

From a Mozilla survey (n=1,410 current or churned open-model developers), the biggest gaps between those still using open and those who churned are operational, not capability:

**Challenges reported by all current/churned open-model developers (ranked):**

| Challenge | % |
|---|---|
| High infrastructure/compute costs | 27% |
| Security, privacy, or compliance concerns | 26% |
| Ongoing maintenance and updates | 24% |
| Complexity of deployment/hosting/scaling | 23% |
| Lack of specialized support | 22% |
| Difficulty evaluating/comparing models | 18% |
| Difficulty fine-tuning/customizing | 18% |
| Difficulty integrating into existing systems | 18% |
| Insufficient documentation/learning resources | 17% |
| Model performance not good enough | 17% |
| No major challenges | 12% |

**By region** (selected highlights): South Asia leans hardest on security (39%) and support (31%); only North America (21%) and Greater China (16%) have >15% reporting no major challenges. Oceania (n=39) and Eastern Europe & CIS (n=98) fall below reliable thresholds. Weighted total sample: 1,411.

---

## Section 2: The Open-Source AI Stack

**Nine layers and 48 components** scored across 10 criteria (1–5 scale). The stack scores high on capability but low on operations.

The report visualizes a heatmap with criteria ordered strongest to weakest. The two coldest columns — **standardization** and **enterprise readiness** — repeat down every layer and component. The report calls this repeating cold edge "the operational gap."

Maturity grades used: Strong (≥4.0) | 3.5–3.9 | 3.0–3.4 | 2.5–2.9 | Weak (<2.5)

Source: Mozilla stack map, June 2026 (48 components, 1,361 projects).

---

## Section 3: Who's Betting on It

### Commercial Scale

Open-weight AI is described as a "multi-hundred-billion-dollar" market with funded companies and enterprise production deployments.

**Key company metrics:**

- **Databricks**: $5.4B run-rate, pre-IPO
- **Mistral AI** (France): ~$400M ARR (20× YoY), ~$14B valuation (talks at €20B), $3.05B disclosed funding. Investors: ASML, a16z, Lightspeed, Nvidia
- **DeepSeek** (China): ~$220M ARR, raised $7.4B at >$50B valuation. Investors: Liang Wenfeng, Tencent, CATL, China National AI Fund
- **Moonshot AI** (China): $3.9B disclosed funding (Meituan/Long-Z, Alibaba, Tencent, HongShan)
- **Zhipu AI** (China): went public via HK IPO 2026 (undisclosed total); prior investors Alibaba, Tencent
- **MiniMax** (China): HK IPO 2026 (undisclosed total)
- **Cohere** (Canada): $1.7B disclosed; open-sourced Command A+ May 2026. Investors: Radical Ventures, Nvidia, AMD, Schwarz Group
- **Cerebras** (USA): $2.1B disclosed (compute layer). Investors: Fidelity, Atreides, G42, Tiger Global
- **Reflection AI** (USA): $2.13B disclosed (open weights). Investors: Nvidia, Disruptive, Sequoia, Lightspeed, DST Global
- **Together AI** (USA): $1.334B disclosed (inference cloud). Investors: Aramco Ventures, General Catalyst, Prosperity7, Nvidia
- **Hugging Face** (USA): $400M disclosed (hub layer). Investors: Salesforce, Google, Nvidia, IBM
- **LangChain** (USA): $260M disclosed (harness tooling). Investors: IVP, Sequoia, Benchmark, CapitalG

Corporate strategics named across the ecosystem: Nvidia, Salesforce, AMD, Google, IBM, ASML, Tencent, CATL, Schwarz Group.

**Five proven revenue models**: hosted inference, enterprise platforms, on-prem licensing, fine-tuning services, and harness tooling.

### The Metered Model Breaks at Scale

On OpenRouter (May–Sep 2025), closed models held ~80% of usage but ~96% of revenue. Closed costs ~6× more per call at ~90% capability parity.

An estimated **~$24.8B in unrealized annual savings** — from the Nagle–Yue study for the Linux Foundation — representing the open-vs-closed price asymmetry.

---

## Section 4: Why It's Happening Everywhere

**More than 70 national AI strategies are live.** The strategic question has shifted from whether to have a national AI policy to which layer of the stack a country can own.

### The Case for Open Is Optionality

The report describes a June 2026 event: three days after Claude Fable 5 went on sale, a single government's export order forced Anthropic to cut access for every foreign national globally. Selective compliance was impossible; models went dark at 5:21 p.m. on a Friday. Anyone who had built on that model inherited a shutdown they had no warning of or part in. The report's conclusion: a provider can switch off a model, but nobody can switch off a copy already running on hardware you control.

**Cloud repatriation context:**
- $90k–$120k to move one petabyte out of AWS S3
- 80% of enterprises now repatriating workloads
- 37signals' cloud bill dropped from $3.2M to under $1M after leaving
- GEICO's cloud costs ran at 2.5× over plan

Closed model APIs are described as reproducing the same trap — building on a proprietary endpoint means inheriting the vendor's pricing changes with no clean exit. Open weights are "exit rights."

### China as the Largest Source of Open Weights

**Cumulative Hugging Face downloads, March 2026:** In February 2026, Qwen out-downloaded the next eight organizations combined. On OpenRouter, Chinese open-weight models rose from under 2% of tokens in late 2024 to more than 45% of weekly traffic by April 2026 (~61% among the ten most-used models). DeepSeek reports 26,000+ enterprise accounts; 58% of new AI startups in 2025 included it in their stack, even as at least eight jurisdictions restricted the hosted service. The resolution: enterprises ban the hosted app and adopt the weights anyway, self-hosted or via Western endpoints.

The report describes this as intentional Chinese policy — the State Council's "AI Plus" Initiative (Aug 2025) and the national Five-Year Plan (Mar 2026) codify open-source proliferation as a core directive. Releasing public weights functions as a macro hedge against semiconductor export controls, offloading global inference onto end users' local hardware. Across the Global South, the draw is diversification away from US technology monopolies; elsewhere it is purely financial. Even Microsoft is described as exploring an Azure-hosted DeepSeek V4 for its heaviest Copilot workload.

**Country-level sovereign investments mentioned:**
- France: €109B
- EU: AI Act GPAI exemptions, four-part package, Frontier AI Grand Challenge, EUROPA consortium
- Portugal: Amália model launched July 2026
- Germany: BMDS SPARK API
- Canada: "AI for All" program; Cohere Command A+ open-sourced; Cohere North Mini Code
- India: 38,231 GPUs, ₹10,372 Cr outlay, 600 data labs, +5M GitHub developers, 13.59% of DeepSeek MAU
- UAE: G42 / $15.2B Microsoft partnership
- South Korea: $71.5B
- Saudi Arabia: Humain $77B / 1.9GW

A world map visualization shows markers scaled to committed public/strategic capital. Source: Open Source AI jurisdictions dataset, July 2026.

---

## Section 5: The Harness Is the New Frontier

The report draws a direct analogy: the browser was the user agent of the open web — code on the user's side negotiating with servers. That role is being recreated one layer up as the **agentic harness** — the orchestration loop, tools, memory, sandboxes, and permission model. This is where production difficulty concentrates and where the open-vs-closed contest restarts.

### Stack Layers (bottom to top):

**Control** (drives the loop):
- Orchestration loop (LangGraph, CrewAI, AutoGen, LlamaIndex) — the reason-and-act cycle
- The model/weights underneath — commoditizing toward zero

**Reach** (connects and remembers):
- Tools & context: MCP
- Agent-to-agent: A2A
- Memory: Mem0, Letta, Zep

**Action** (does things safely):
- Sandboxes & execution: E2B, Daytona, Modal
- Permission & identity — "the write surface — the unsolved gap"
- Eval & observability: Langfuse, Phoenix

**Surface** (meets user & money):
- Interface: AG-UI, A2UI
- Payment & metering: x402, AP2, UCP

**Govern** (one plane over many harnesses):
- Stateful policy (what the session already did)
- Registry & lineage (which agent did what)
- Budget & revocation (cost caps, kill switch)
- Meta-harness tools: Databricks' Omnigent, OPA, Agent governance toolkit

### Adoption Metrics for the Harness Layer

- LangChain: 126,000+ GitHub stars, 60% developer share
- MCP: 97M monthly SDK downloads, 10,000+ active servers in its first year, 4,750% growth in 16 months, donated to the Linux Foundation's Agentic AI Foundation in December 2025
- Only ~21% of companies report mature agent governance

### The Model Is Eating the Harness

**Terminal-Bench 2.0 (May 2026):** A third-party scaffold ran Anthropic's own weights to 79.8% while Claude Code managed 58.0% on the same model — a 21.8-point spread putting the harness ahead of the weights.

**Terminal-Bench 2.1 (July 2026):** Frontier labs pulled the harness in-house. On every model where both appear, the lab's own harness now wins. The 21.8-point gap compressed to roughly 3 at the top. The model is "eating its way up the stack," weights and scaffold shipped as one product.

**The moat in formation:** A harness tuned tightly to one lab's weights becomes a fit rather than a neutral layer. It degrades on anyone else's model. Lock-in arrives as a side effect of optimization. Open models have no first-party harness to answer with — none appear in the verified top tier of the official Terminal-Bench 2.1 board.

**But on a neutral scaffold (vals.ai's Terminus-2 run):** The capability gap collapses to a few points; the price gap is 5×. The strongest open model (GLM 5.2) lands a fraction behind Claude Opus 4.7 and ~4 points behind Opus 4.8 at roughly one-fifth the cost.

**Data flywheel:** Usage routed through a lab's harness feeds back into its next model. That edge is real — and it cuts both ways: the usage exhaust trains whoever owns the harness.

### The Write Surface — The Unsolved Permission Problem

**Reads:** Reversible, low-consequence (fetching a document, querying a database). Largely permitted by default.

**Writes:** Costly or irreversible side effects (sending a message, spending against a budget, modifying a record, executing a transaction). Where confirmation, approval thresholds, cost caps, and revocation must concentrate.

No portable model defines which writes an agent may perform unattended, which require human approval, and which are forbidden — across MCP hosts, A2A peers, direct tool invocations, and framework boundaries. MCP's 2025-11-25 specification moved authorization onto OAuth 2.1; A2A v1.0 standardized signed Agent Cards — both stop at authentication. Knowing who an agent is says nothing about what it may do.

**Consent fatigue** is identified as a top-tier threat in CoSAI's MCP threat model — users approve the large majority of prompts. This is itself a write-side failure.

**Emerging cross-harness architectures** (like Databricks' open-sourced Omnigent) enforce stateful, contextual policies that track session history and gate writes accordingly — requiring human approval after an agent has pulled an unverified package, or enforcing cost caps that pause a session. These govern from a layer above any single harness.

### Closed Is Not the Same as Secure

Closed API safety (filtering, monitoring, revocation) comes from the serving layer, not from keeping weights secret. The same controls live at the harness layer for self-hosted open models. In 2025, authorization failures rated CVSS 9.3–9.4 hit Anthropic, Microsoft, ServiceNow, and Salesforce — all closed systems. The NTIA studied whether to restrict open weights and recommended monitoring instead.

### Where Closed Still Leads (four areas):

1. **Integrated harness** — no open model on the verified top tier of Terminal-Bench 2.1's official board; even on a neutral scaffold the best open model trails Opus 4.8 by ~4 points, and the data flywheel behind the lab harness feeds the next model
2. **Long-context fidelity at 1M tokens** — Gemini 3 holds 89% multi-needle retrieval vs. DeepSeek V4-Pro's 41%
3. **Turnkey compliance** — SOC 2, HIPAA, zero data retention by default
4. **Accountability** — a counterparty the customer can hold liable

The report notes that compliance and accountability are contracting problems; the integrated harness is a tooling problem; long-context fidelity is a model problem only the open labs can solve.

---

## Section 6: Opportunities

"Five bets. None requires beating the frontier." They require owning the layers above it — the harness, memory, and permission model — while those layers are still open. (The five bets are named but not individually elaborated in the scraped content.)

---

## Section 7: The Watchlist

Four signal categories with reversal conditions:

### Capability & Adoption
The 3.3% gap (parity on coding, behind on reasoning and agentic), and open's OpenRouter token share especially in agentic coding.
**Reverses if:** token share stalls while the reasoning gap widens.

### The Harness
Terminal-Bench spread between lab-owned and independent scaffolds; MCP/A2A governance under the AAIF; the portable permission spec that doesn't yet exist.
**Reverses if:** the lab-harness lead widens, or a closed platform sets the permission standard first.

### Market Structure
Open-lab economics (ARR, raises, Zhipu/MiniMax IPOs) against metered-pricing breakpoints (~2027–28), with sovereign capacity as counterweight.
**Reverses if:** sovereign funding lapses or open-lab economics fail to scale.

### Trust & Safety
Tracked, not settled: misuse capability and how easily safety tuning strips from open weights; hard-friction zones (synthetic CSAM and NCII); whether NTIA's "monitor, don't restrict" holds.
**Reverses if:** a major misuse event or a shift from monitoring to restriction.

The report closes: "There is a test you can run for the rest of this. Look at who is seated in the rooms where AI gets decided, and with what status. The day they seat the people who keep AI open, portable, and widely deployed on equal footing, the shift from renting to owning will have happened."

---

## Closing

This is V1. Contact channels provided: email (opensource@mozilla.org), X, LinkedIn, Bluesky. Sign-up link for MozFest.

Report dated July 2026, last updated June 30, 2026. Described as "A recurring assessment of the open model ecosystem."

---

## Citations (full bibliography preserved)

### Section 1
Sources cited include: Stanford HAI AI Index 2025 and 2026 reports; Chatbot Arena data; OpenRouter 100T-token study and live leaderboard; FT analysis; Mistral financial reporting; Epoch AI inference price trends; MIT study (Nov 2025, arXiv); DeepSeek-R1 model card; Analytics Vidhya o1 pricing comparison; Tony Blair Institute; Fortune/NANDA MIT study; Stanford enterprise AI playbook.

### Section 2
Source: Mozilla stack map, June 2026 (48 components, 1,361 projects).

### Section 3
Sources include: Databricks revenue reporting; Reuters; Sacra/Mistral financials; TechCrunch Mistral valuation; ElectroIQ DeepSeek statistics; WSJ DeepSeek funding; TechBuzz AI; Cybernews; Fortune Uber AI spending; Yahoo Finance; TechCrunch Uber caps; Introl/vLLM; Axios Microsoft Copilot; Linux Foundation open-model economics.

### Section 4
Sources include: Hivelocity AWS egress; Byteiota cloud repatriation; The Register 37signals; HBS GEICO; Anthropic Fable access announcement; arXiv Qwen paper; SCMP Qwen downloads; Digital Applied OpenRouter rankings; TechTimes; Click-Vision DeepSeek stats; Sustainable Tech Partner restricted jurisdictions; CSET State Council analysis; BISI Five-Year Plan; USCC export controls analysis; Élysée France AI plan; EU Commission press releases; PitchBook; Reuters Portugal Amália; BMDS Germany; PM Canada AI for All; BetaKit Cohere; IndiaAI compute; Economic Times GitHub India; Thunderbit; OECD.AI; Oxford Insights; Microsoft G42; Fortune Business Insights South Korea; Introl Saudi Humain.

### Section 5
Sources include: MHTechIn LangChain; Pento.ai MCP year in review; Databricks Omnigent blog; Terminal-Bench 2.0 and 2.1 leaderboards; Vals.ai Terminus-2 runs; VentureBeat GLM 5.2; Anthropic MCP donation/Linux Foundation AAIF; MCP authorization specification; A2A specification; Intuition Labs; Agent Market Cap memory landscape; Modal sandboxes; Firecrawl observability; Marktechpost auth platforms; CNAS NTIA response; Okta CVSS failures; NTIA monitoring recommendation; Digital Applied long-context retrieval; Future of Life AI Safety Index; ITPro Linux Foundation contractual assurances.

### Section 6 (Opportunities)
Sources: OpenAI "Hundreds" post; Anthropic Series H announcement.

---

*No sponsor or funder information beyond Mozilla is disclosed in the page content.*
