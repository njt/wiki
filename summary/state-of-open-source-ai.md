---
url: https://stateofopensource.ai/
title: "The State of Open Source AI — V1.0"
author: Mozilla (Raffi Krikorian, CTO)
date_fetched: 2026-07-18
date_published: 2026-07
topics:
  - ai-research-and-models
---

Mozilla's inaugural assessment of the open-source AI ecosystem, published July 2026. It maps capability gaps, adoption economics, the operational tooling stack, sovereign investment, and the emerging agentic harness layer — arguing that open weights have reached near-parity on coding and general tasks while trailing on reasoning, and that the next contest is over the harness (orchestration, memory, permissions) rather than the model itself.

The headline: open-weight models closed the Chatbot Arena gap from 8% to roughly 0% on coding and instruction-following by mid-2025, though it reopened to ~3.3% as closed reasoning models pulled ahead. Meanwhile GPT-4-class inference costs collapsed ~50× in 36 months, and open-weight models now route a majority of tokens on OpenRouter — concentrated in coding and agentic workloads. Chinese-built open models account for roughly three times the weekly token volume of US-built ones. Mozilla's own developer survey finds 79% of teams adding AI use open models, but only 51% of those teams reach production (vs. 63% for closed), a gap the report attributes to operational tooling rather than model capability.

The report scores 48 components across nine layers of the open AI stack on ten criteria. The two coldest columns — standardization and enterprise readiness — repeat across every layer, which the report calls "the operational gap."

A central argument is that closed model APIs reproduce the vendor lock-in of the cloud era: a provider can switch off access (as happened when Anthropic cut foreign-national access to Fable 5 in June 2026), but nobody can switch off a copy already running on hardware you control. Open weights are framed as "exit rights." The report also documents China's emergence as the largest source of open weights — driven by state policy that treats public model releases as a macro hedge against semiconductor export controls — and catalogs sovereign AI investments across France (€109B), South Korea ($71.5B), India, Saudi Arabia, and others.

On the harness: the report draws a direct analogy to the browser's role in the open web, arguing that the orchestration loop, tools, memory, sandboxes, and permission model are where production difficulty concentrates and where the open-vs-closed contest restarts. Terminal-Bench 2.1 shows frontier labs shipping models and harnesses as one tuned product — closing a 21.8-point gap to roughly 3 points — while no open model appears on the verified top tier. The unsolved permission problem (the "write surface") is identified as a critical gap: no portable spec defines what an agent may do unattended across frameworks.

Four areas where closed still leads: integrated harness, long-context fidelity at 1M tokens, turnkey compliance, and contractual accountability. The report closes with five opportunities (none requiring beating the frontier), a watchlist of four signal categories with reversal conditions, and a challenge: look at who has a seat where AI decisions are made.

---
*Sources: [[raw/state-of-open-source-ai]]*
*Last updated: 2026-08-01*
