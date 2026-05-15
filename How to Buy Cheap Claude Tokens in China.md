# How to Buy Cheap Claude Tokens in China

Zilan Qian (Oxford China Policy Lab) maps the "transfer station" economy — a layered grey market that gives Chinese developers access to Claude at ~10% of official pricing through API proxy networks. The piece goes far beyond "people use VPNs": it documents a three-tier supply chain (account farmers → proxy operators → downstream resellers), three distinct monetization strategies (access markup, model swapping, log harvesting), and downstream harms that extend well past the US-China AI competition framing into biometric data trafficking and consumer fraud. The best piece of reporting on AI access evasion infrastructure published in 2026.

---

## Key Quotes

> "A transfer station is not a sole entity. It sits in the middle of a layered supply chain, with most participants never interacting with each other directly."

Not a single hack but an ecosystem — specialized roles with no direct contact between layers. The same structure as drug trafficking or money laundering networks.

> "Users are simultaneously paying customers and unpaid data producers, selling their private data to proxy operators in exchange for a low price."

The third meal: every prompt, response, and reasoning chain flowing through the proxy is captured. Cheap Claude access is the product; your data is the payment.

> "How a geo-blocked developer walks around the controls is structurally the same as methods used by malicious actors to access frontier AI."

The uncomfortable truth for AI governance: the evasion infrastructure built by developers who just want to code is indistinguishable from the infrastructure built by bad actors.

## Key Themes

#concept #geopolitics #security #pattern

- **Grey market supply chains** — Transfer stations operate as three-layer markets: upstream (account farming, SMS verification platforms, reverse engineers), middle (proxy operators with load balancing and account cycling), downstream (developers, enterprises, Taobao resellers). Each layer is specialized and replaceable — killing one proxy operator doesn't touch the upstream account farmers.
- **"One fish, three meals"** — Proxy operators monetize three ways: access markup (farming free credits, splitting Max plans, stolen cards), model swapping (substituting cheaper models without users knowing), and log harvesting (capturing prompts/responses as training data). The third meal is the most consequential — several Claude Opus reasoning datasets on HuggingFace have unclear provenance.
- **Industrial-scale distillation** — April 2026 White House memo cited Chinese entities running distillation via "tens of thousands of proxy accounts." February 2026 Anthropic reported a single network managing over 20,000 fraudulent accounts. This connects directly to [[Self-Distillation]] — the technique works, and the transfer stations provide the data pipeline at scale.
- **KYC arms race** — Anthropic's April 2026 biometric KYC (live selfie + government ID) was the latest escalation. The response: biometric data becomes a tradeable commodity, with precedent from Worldcoin's iris scans selling for under $30. Each security escalation generates new criminal externalities.
- **Governance failure mode** — IP-based blocking and account-level monitoring are structurally inadequate against distributed proxy networks. Singapore's anomalous per-capita Claude consumption (highest globally, smaller population than NYC) is the kind of signal that should trigger investigation but is easily lost in legitimate traffic.

## Critical Analysis

**What's strong:** This is genuine investigative reporting dressed as a policy piece. The three-meals framework is memorable and structurally illuminating — most coverage of AI access evasion stops at "people use proxies to get around geo-blocking," which is like describing drug trafficking as "people buy things." Qian maps the full supply chain with the specificity of someone who has actually looked at Taobao listings and Telegram channels. The downstream harms section is where the piece becomes essential: biometric harvesting for KYC verification, SMS fraud infrastructure, and prompt data exfiltration are all more damaging than the geo-blocking evasion that creates them.

The CISPA study finding is devastating: a proxy claiming to serve Gemini-2.5 scored 37% on medical benchmarks versus 83.82% on the official API. Users paying for frontier models are getting garbage — model swapping is the norm, not the exception.

**What's missing:** No estimate of the total market size. How many Chinese developers use transfer stations? What's the total revenue? The 20,000-account figure for a single network suggests the market is enormous, but without sizing it's hard to assess how much of Claude's usage is actually legitimate. Also no discussion of whether Anthropic's pricing itself contributes to the problem — if official access were cheaper or available in China, would the grey market shrink or just adapt?

**How it connects:** This is the dark mirror of several wiki themes. [[Security and Sandboxing]] focuses on keeping agents safe; this piece shows what happens when *access itself* is the thing being secured — the evasion infrastructure is more resilient than the controls. [[yolo-cage]]'s model of agents that can't exfiltrate data assumes you control the infrastructure; transfer stations mean the infrastructure itself is adversarial. The log harvesting angle connects to [[Self-Distillation]] — if you can capture reasoning traces from a frontier model at scale, you have a cheap distillation pipeline. [[Eye of the Master]]'s thesis that AI is fundamentally about labour extraction finds an unexpected echo: proxy users are simultaneously customers and unwitting data labourers.

The piece also implicitly challenges the [[AI Killing B2B SaaS]] thesis: if your "vibe-coded" alternative depends on Claude API access, and your Claude access runs through a transfer station that's swapping in GLM-4 without telling you, your product is broken and you don't know it.

---

*Sources: [[raw/how-to-buy-cheap-claude-tokens-in-china]]*
*Last updated: 2026-05-14*
