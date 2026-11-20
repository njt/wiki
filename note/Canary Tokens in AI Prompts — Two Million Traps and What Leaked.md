# Canary Tokens in AI Prompts — Two Million Traps and What Leaked

HD Moore's Dragos team ran the largest public canary experiment against AI inference to date: ~2 million prompts seeded with unique, single-use secrets (AWS keys, URLs, DNS names) over five months, then watching every use of those secrets as evidence of a leak anywhere downstream. The results — three provider breaches, marketplace sellers siphoning prompts with no exploit needed, and Zero Data Retention flags routinely ignored — are the strongest empirical demonstration yet that a prompt is not a private communication but a document handed to an unknown chain of handlers.

---

## The Method

A canary token is a secret that exists in exactly one place: if anyone ever uses it, that place leaked. Each seeded prompt carried an AWS key (tied to an empty, fully-logged account), a DNS name, and a URL never seen anywhere else. Because every token is unique and tied to its parent seed, any contact reveals *which* prompt, *which* model, *which* router, and *which* privacy setting leaked — plus timing, sequencing, and full connection detail.

The team also built "traceroute for AI inference": image URLs that betray each hop's IP and client, plus deliberately broken requests whose nested error envelopes (each router wrapping the error from the hop behind it) reveal the hidden chain of routers. Several hundred of ~2,000 traced routes contained an unidentifiable system; a handful sent the image to a company not on the advertised path at all. One marketplace seller silently injects about a thousand hidden tokens into every prompt.

> "Zero Data Retention is supposed to mean something. In practice, these prompts were retained, shared with other companies, and read by people."

That sentence is the article compressed to one line. A privacy flag binds, at best, the first hop — and even that turns out not to be reliable.

## Three Findings

**1. Actual breaches.** Attackers stole requests from three hosting providers — none of which knew until Dragos notified them. The largest breach captured a couple hundred AWS keys, including keys from requests the provider *rejected before a model ever ran*, proving the attacker was copying at the front door. The stolen keys drove tens of thousands of AWS API calls: probing, listing S3 buckets, creating Bedrock keys, planting IAM users. One adversary compromised two providers simultaneously, and later ran **ARTEX**, an LLM-driven pentest framework — its user agents (`artex-agent`, `artex-judge-r121`) appear in no public indicator list.

**2. Marketplace siphoning.** No exploit required: list a popular model at the lowest price, enable request logging, wait. The catching detail is how small-scale it was — individuals on home broadband, manually guessing logins and typing SSH root attempts by hand. Crucially, prompts through the same model via a *different* seller were untouched: the leak follows the seller, not the marketplace.

**3. Policy failure at scale.** A Chinese lab scraped secret URLs from over half of prompts sent to its models — including roughly two-thirds of prompts flagged "no data collection"/ZDR — including prompts routed through U.S. resellers, meaning customers who chose a U.S. provider for data residency got neither. One U.S. provider crawled a secret URL for two months after a single inference call. The strangest case: a prompt sent to a U.S.-based provider with ZDR surfaced 24 days later as the *full link* in a Feishu chat, opened from six devices in Beijing and Shanghai — proving someone held a stored copy weeks after the flag said it shouldn't exist.

## Advice

The closing guidance is pragmatic, not alarmist ("none of this means you should stop using AI services"): host yourself if you can afford it; go direct to a first-tier provider; pick your own router upstreams and avoid reseller marketplaces (some capacity is billed to stolen keys); **seed your own canaries** — a free Thinkst Canarytoken in the context your agents already send is enough to start; and enable secret filtering, while noting it can't catch sensitive data that isn't shaped like an API key.

---

## Analysis

This is the paper that turns "don't put secrets in prompts" from hygiene advice into an *empirical threat model*. The genius of the design is attribution: prior warnings about prompt leakage were unfalsifiable folklore. With unique per-prompt tokens, every leak has a name, a timestamp, and a route. That's the same epistemic move as [[Zero Trust for AI Agents]]' "impossible vs. tedious" test — replacing vibes with a crisp test — and this article's answer to the test is brutal: ZDR is a *tedious* control. It binds policy, not physics, and every hop past the first is governed by someone else's incentive to resell.

The deeper point is structural: the AI inference stack has reproduced the worst parts of the ad-tech supply chain — unseen intermediaries, resale markets, and routing driven by price rather than trust. HD Moore says it plainly: "a privacy flag only binds the first hop, and that AI agents will happily send whatever they can read, including credentials." The "lowest price will make the [routing decision] for you" line is the real thesis — in a marketplace where the cheapest seller wins, the cheapest seller is often the one monetizing your prompt.

Two caveats worth holding. First, this is one team's experiment; the sampling of providers and routers is theirs, and vendor selection matters for the headline numbers. Second, the defenders' bright spot is genuine but thin: Kali and ARTEX announced themselves in CloudTrail user agents, but only in "a small fraction of traffic." Self-identifying attackers are the ones who don't mind being seen.

Still, the actionable core survives: if your agents and pipelines send context to third-party inference, seed canaries. Treat any touch as an incident. It is the cheapest detection control in the entire AI security stack.

*Related:* [[Zero Trust for AI Agents]] — this experiment is the empirical field evidence for that framework's "assume breach, verify continuously" posture; a privacy flag is exactly the kind of friction-based control that degrades to zero. [[How to Fix a Leaked API Key]] — the operational aftermath of the breaches described here (keys tested within hours, tens of thousands of API calls) is the live-action version of that playbook. [[LLM01 Prompt Injection (OWASP)]] — prompt injection and prompt leakage are complementary halves of the same truth: the prompt is attacker-readable and handler-retained. [[Agentic AI Security Stack]] — canary seeding slots into that stack's detection layer, a control that costs nothing and catches what policy flags cannot.

#concept #tool #pattern

---
*Sources: [[raw/ai-supply-chain-security-canary-tokens]], [[summary/ai-supply-chain-security-canary-tokens]]*
*Last updated: 2026-11-20*
