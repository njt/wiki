# Security Is Hard, Y'all

Eric Lawrence's first-person account of being phished by a legitimate Cloudflare product — and what that reveals about the structural impossibility of web security UX. When a real product from a company you trust hits every checkmark on the "this is a phishing attack" list, the failure isn't the user's. It's the system's.

---

## The Incident

Lawrence, a longtime Cloudflare user and former Microsoft Edge security engineer, saw a tweet about Cloudflare Wallet — a new product for registering payment handles. He clicked through to `cloudflare.pay`, found his preferred handle `@ericlaw` available, and signed in via OAuth to claim it. Then the authorization screen appeared, and his training kicked in.

> "This looks **exactly** like one of those Consent Phishing attacks that have been so popular over the last few years!"

The red flags cascaded: the product lived on `cloudflare.pay`, a domain with no inherent technical relationship to `cloudflare.com`. The `.pay` sTLD is available to anyone with $20 — unlike `.bank`, which requires vetting. The OAuth consent screen didn't recognize its own company's feature. A green checkmark looked like an emoji an attacker would stuff into a misleading display name, the same technique used to phish Microsoft email accounts. The Cloudflare dashboard had no link to the new product. Documentation searches turned up nothing. The AI chat agent asked for full account access before answering questions.

> "The guys at Cloudflare are geniuses who know their stuff. This has **got** to be an attack."

He reported it as a phish. It wasn't. It was a legitimate Cloudflare product launch.

## Why This Matters Beyond the Anecdote

This isn't a "silly user" story. Lawrence is a security professional who spent years building browser security features. He identified every signal correctly — they *were* the same signals that indicate phishing attacks. The system failed, not the human.

> "When legitimate websites sometimes act very very phishy, consider how hard it must be for URL Reputation services like Microsoft SmartScreen and Google SafeBrowsing to block malicious sites without false positives as millions of new sites are added to the web every week."

The structural insight: **every anti-phishing signal is a heuristic, not a proof**. New domains, unfamiliar TLDs, OAuth consent screens, missing documentation, green checkmarks in display names — attackers use all of these, but so do legitimate product launches. URL reputation services walk a knife edge between blocking real attacks and breaking real products. There is no signal that cleanly separates them.

## The Trust Infrastructure That Failed

### Domain Names as Trust Anchors

The `cloudflare.pay` domain is the central failure. Users have been trained (correctly) to trust `cloudflare.com`. A `.pay` domain has no cryptographic or organizational link to a `.com` domain — it's just another $20 registration. Lawrence's recommendation is blunt and correct:

> "Host apps and content under your trusted domain name (e.g. `cloudflare.com/pay` or `pay.cloudflare.com`). If you must add a new name, link to it directly from a page on your trusted domain name."

This is the same principle at the heart of [[Hostnames and Usernames to Reserve]]: protocol trust models assume deliberate, manual naming. When companies fragment their domain surface across arbitrary TLDs, they break the only trust anchor users have.

### Security UI Placement

The green checkmark that looked like a phishing emoji was actually a legitimate security indicator — poorly placed so that users had to hover over it to see the details. This is a failure of security UX design, not user attention. Security information displayed in untrustworthy locations becomes anti-signal: it trains users to ignore it.

### The Principle of Least Privilege, Ignored

The AI chat agent's first proposal was **full control** rather than **read only** access. Lawrence notes this "feels like a failure of the principle of least privilege" — and he's right. The same principle is central to [[Zero Trust for AI Agents]]'s concept of Least Agency: tools should default to minimal capability and escalate only on demonstrated need.

### Reporting Infrastructure

There was no "Report suspicious request" link on the permission page. Cloudflare's preferred reporting channel (HackerOne) had a broken CAPTCHA that prevented login. The trust-and-safety infrastructure that should have caught this incident was itself broken.

> "Make it trivial to report scams, in context (e.g. on the permission request page). Test your security reporting flows to ensure they are monitored and function correctly."

## The Broader Security Lesson

Lawrence's post is really about the **structural asymmetry** of web security:

- **Attackers** have every incentive to look legitimate and can iterate rapidly.
- **Legitimate products** launch on new domains with unfamiliar flows and no track record.
- **Users** have only heuristics — domain names, TLS indicators, UI patterns — that both sides can replicate.
- **URL reputation services** must block attacks without false positives, despite attackers and legitimate launches looking increasingly identical.

This asymmetry is the reason "never blame the victim" isn't just compassion — it's systems thinking. The signals users are told to trust are structurally unreliable.

## Lessons for Developers

Lawrence's recommendations, lightly annotated:

1. **Host under your trusted domain.** `pay.cloudflare.com` would have prevented the entire confusion. New TLDs destroy the trust users place in your primary domain. See also [[Hostnames and Usernames to Reserve]] on why namespace trust is protocol-level infrastructure, not marketing.

2. **Show security information in trustworthy places.** A green checkmark floating ambiguously in a consent dialog is indistinguishable from an attacker's emoji. Security UI must be anchored to chrome the attacker can't replicate.

3. **Make scam reporting trivial and in-context.** Every permission screen should have a "Report this" link. Reporting flows should be tested as rigorously as payment flows.

4. **Test your security reporting flows.** Cloudflare's HackerOne CAPTCHA was broken. A reporting channel that doesn't work is worse than no channel — it creates the illusion of coverage.

5. **Default to least privilege.** The AI agent asking for full control when read-only would suffice is a cultural failure, not a technical one. See [[Zero Trust for AI Agents]] for the operational framework.

## Critical Analysis

**What's strongest:** The first-person narrative is devastating because Lawrence *did everything right*. He identified the phishing signals, checked the dashboard, searched docs, asked the AI agent, and reported the phish. At every step, the system confirmed his suspicion. This isn't a story about user error — it's a story about a system that generates false positives for its own legitimate products.

**The Cloudflare irony:** Lawrence now works at Cloudflare. The fact that an internal security expert couldn't distinguish a company product from an attack against the company is a damning indictment of the product launch process. The post doesn't say this explicitly — it doesn't need to. The facts say it.

**What's underdeveloped:** The post doesn't explore the economic incentives that lead companies to launch on new domains. Marketing wants `wallet.pay` because it's clean branding. Security wants `pay.cloudflare.com` because it's trustworthy. Nobody in the product launch pipeline has "will this look like a phishing attack?" as a gating criterion. That's a process failure, not a technical one.

**The reputation service angle:** Lawrence's point about SmartScreen and SafeBrowsing is the most important and least developed. These services *must* block `cloudflare.pay` if it were an attacker — the signals are identical. They *must not* block it when it's legitimate. They are structurally required to solve a problem with no solution. The fact that they work as well as they do is remarkable; the fact that they fail is inevitable.

**Connection to [[Zero Trust for AI Agents]]:** The "impossible vs. tedious" test applies here with force. Every anti-phishing signal discussed in this post — domain name, TLS certificate, green checkmark, OAuth consent screen — is in the "tedious" category. An attacker can replicate all of them. None of them make phishing impossible. The only defense is a hard barrier (like WebAuthn/phishing-resistant MFA), and even that doesn't help when the legitimate product itself looks like an attack.

**What's missing:** No discussion of WebAuthn, passkeys, or phishing-resistant authentication as structural solutions. No mention of the role app stores and curated platforms play in reducing this ambiguity (at the cost of centralization). No exploration of whether the `.pay` TLD itself is part of the problem — a TLD with no identity vetting that sounds like it should have identity vetting.

---

## Key Themes

#security #phishing #ux #trust #domain-names #consent-phishing #url-reputation #usable-security #concept

---

## See Also

- [[Hostnames and Usernames to Reserve]] — Namespace trust as protocol-level infrastructure; the same principle applied to domain fragmentation
- [[Zero Trust for AI Agents]] — The "impossible vs. tedious" test applied to anti-phishing signals
- [[Security and Sandboxing]] — The usability-vs-security tension that this incident embodies
- [[Cloudflare Temporary Accounts for Agents]] — Cloudflare's agent-native infrastructure design; the contrast with this product launch is instructive
- [[An Illustrated Guide to OAuth]] — The consent screen flow that triggered Lawrence's phishing detection
- [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] — The principle the AI chat agent violated by defaulting to full control
- [[PACT Anonymous Credentials for the Web]] — Cryptographic approaches to trust without identity, an alternative to the heuristic-based trust this article critiques
- [[DKIM2 and DMARCbis]] — Email authentication's parallel problem: protocol trust assumptions that break at scale

---
*Sources: [[raw/security-is-hard-yall]], [[summary/security-is-hard-yall]]*
*Last updated: 2026-08-06*
