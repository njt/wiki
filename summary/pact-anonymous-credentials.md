---
url: https://hacks.mozilla.org/2026/06/pact-anonymous-credentials-for-the-web/
title: "PACT: Anonymous Credentials for the Web"
author: Dennis Jackson (Senior Staff Cryptography Engineer, Mozilla)
date_published: 2026-06-23
date_fetched: 2026-06-24
publication: Mozilla Hacks
topics:
  - ideas-and-culture
---

# PACT: Anonymous Credentials for the Web

Dennis Jackson, Senior Staff Cryptography Engineer at Mozilla, lays out the technical architecture for PACT (Private Access Control Tokens), a proposed open standard for anonymous rate-limiting credentials on the web. This is the technical companion to Mozilla's Distilled post "Keeping the web open and private in the bot era."

## The Problem

Browsing with privacy protections (private windows, VPNs, anti-fingerprinting) increasingly triggers "registration walls, block pages, and endless CAPTCHAs." Websites need to block bots engaged in volumetric abuse — "SEO comment spam, credential stuffing and DDoSing" — but the tools for doing so are breaking.

Browser privacy protections are "dismantling the passive signals that anti-abuse systems depended on," while generative AI has made CAPTCHAs ineffective — bots solve them "faster and more reliably than humans." Sites respond by demanding identifying information (email, federated login, disabling VPNs), creating greater friction and enabling cross-site tracking.

The core dynamic: "The more effectively they protect their privacy, the harder it is for websites to distinguish them from bots and the worse the treatment they receive." Browser-based AI agents intensify the tension, as sites cannot distinguish legitimate agent traffic from volumetric abuse.

## Critique of Existing Solutions

**Google's Web Environment Integrity** (abandoned 2023) is "the blunt version." It attested to the user agent, OS, and device. Users lose control to the attester (which decides what gets blessed) and to the website (which decides what to accept). If sites adopted allow-lists, "building a new browser would have become virtually impossible."

**Apple's Private Access Tokens** (deployed 2022), built on the IETF-standardized Privacy Pass protocol, "get a lot right" — one-time tokens prevent linking visits. However, they rely on device attestation requiring hardware manufacturer control. "There's no way to open the system to other sources of scarcity without compromising the system's privacy properties," meaning access could become "tied to having bought expensive hardware from a small, hard to change set of vendors."

Both approaches are "ultimately hostile to users and to the openness of the web."

## The Core Insight: Rate Limiting Over Identity

"Bots' harms arise from their ability to operate beyond human scale." Sites don't need user identity or proof of approved software — they need to know visitors are restricted to a site-set rate limit.

Rate limits require anchoring to "something scarce; something an attacker can't cheaply replicate." Alternatives to hardware scarcity: "email addresses and phone numbers are naturally scarce. A paid subscription costs an attacker the same as a real user."

The vision: "an open ecosystem with many parties offering scarcity signals, each site choosing which to accept." VPN providers are a concrete example — a VPN subscription is a source of scarcity, and if providers could vouch for users, those users could browse with less friction.

The core difficulty: the system must take information from one site (that a user holds something scarce) and expose it to other sites for rate limiting, without enabling cross-site tracking. The goal: "no more than the minimum information gets through: a single bit communicating whether the user is below the rate limit set by the site."

## Cryptographic Foundations

**Privacy Pass** (originally developed 2018 for Cloudflare CAPTCHAs with Tor) provides the core primitive — tokens that are "unlinkable between issuance and redemption." A user proves something to an issuer, receives tokens, and later presents one; the website verifies legitimacy but cannot link the token back to the user.

Current deployments: Apple uses it for Private Cloud Compute and iCloud Private Relay authentication, Chrome for two-hop IP protection, and Kagi for private search.

Challenges in moving to an open system:
1. **Privacy leak via issuer identity** — learning which issuer a user has tokens from reveals their relationship with that site, creating a fingerprinting vector.
2. **Issuer blinding** — zero-knowledge techniques let a client prove it has a token from a set of acceptable issuers without revealing which one. Generic approaches are slow, but "bespoke approaches tailored to the underlying cryptography can improve this considerably."
3. **Trust and accountability** — if sites can't see which issuer a user used, detecting misbehaving issuers becomes difficult. Sites need aggregate scoring per issuer via multiparty computation systems like **Prio**.
4. **Dynamic rate adjustment** — tokens are hard to invalidate mid-stream.

**Anonymous Credit Tokens (ACT)** address the dynamic adjustment problem. ACT enables "the use of a credential with state" — an internal counter that a site can verify exceeds a threshold and mutate (increment or decrement) based on behavior. "The exact value is never leaked to the site," preventing tracking, and successive presentations of the same credential "can't be linked."

## PACT Architecture

PACT — **Private Access Control Tokens** — was first sketched at a May 2026 W3C workshop in collaboration with Cloudflare, Chrome, and other stakeholders.

**Three core roles:**

- **Anchor** — provides a source of scarcity and issues **Endorsement tokens** (following Privacy Pass) to users who meet criteria (subscription, good-standing account, verified phone number). In practice, Anchors could be any website with such signals.

- **Moderator** — handles rate limiting for a site. Verifies Endorsements and issues **Credentials** (stateful objects using ACT). Each site nominates a single Moderator; commonly the site itself fills this role. A Moderator can also be a third-party shared across sites for cooperative rate limits.

- **Client** (browser/user agent) — carries Endorsements and Credentials.

**Issuer blinding** ensures that when a Moderator redeems an Endorsement, "it only learns that it came from one of the Anchors it trusts, but not which one."

**Three protocol flows:**

1. **Anchor Flow**: Users receive Endorsements through normal browsing at sites they have relationships with. This is intentionally "a relatively rare operation" — Endorsements should not be too easy to accumulate.

2. **Endorsement-to-Credential Exchange**: When a user visits a site using a Moderator, the browser spends an Endorsement from a trusted Anchor and receives a Credential in return. The Anchor's identity is hidden from the Moderator. If the user has no suitable Endorsements, existing mechanisms (CAPTCHAs, account creation, federated login) bootstrap a Credential — "the system degrades to today's experience rather than locking the user out."

3. **Credential Presentation & Update**: As the user browses, the browser presents the Credential. The Moderator updates its internal state — rewarding benign behavior, penalizing suspicious activity — but "can't track the use of the Credential or identify it if it's used on other sites the Moderator covers." Revocation happens naturally: the Moderator simply refuses to return an updated Credential.

## AI Agents

AI agents "acting on behalf of a user slot into the same flow." An agent can carry its user's Credentials, keeping the user accountable. Alternatively, an agent operator can run its own Anchor. "Sites retain control over which Anchors they accept, so they can choose how to treat agent traffic without needing a separate detection mechanism."

## Privacy Properties

Multiple mechanisms combine to limit information leakage to "close to a single bit":
- Cryptographic unlinkability prevents Credential presentations from being tied to each other or to original issuance.
- Each site has a single Moderator, preventing the set of Moderators from becoming a cross-site fingerprint.
- The Anchor-to-Credential exchange happens in an isolated browsing context.
- When the Moderator updates a Credential, it adjusts the state "without learning what it is."

## Trade-off: Uniform Initial Access

Because the Moderator "can't see which Anchor backed a Credential at issuance," it can't differentiate initial access between strong and weak Anchors — doing so would leak which Anchor was used. The initial access must be uniform across the Moderator's Anchor pool, "in practice meaning setting it at the strength of the weakest." Credentials accrue access over time based on behavior.

## Aggregate Scoring

To enable effective Anchor trust decisions, users can present "an encrypted share which identifies the anchor they used" that is privately aggregated via MPC (like Prio) to compute issuer quality.

## Next Steps

The architecture "has the right shape, but many of the details still need to be worked out." The IETF is the intended venue for cryptographic protocol specifications, and the W3C for the WebAPI surface. Draft specifications will be brought to these bodies. The post welcomes "collaborators from across the ecosystem: browser vendors, site operators, anti-abuse providers, and the cryptography community."

## Acknowledgements

Collaborators: Watson Ladd, Thibault Meunier, Michele Orrù, Trevor Perrin, Eric Rescorla, Samuel Schlesinger, Martin Thomson, Eric Trouton, Benjamin Vandersloot, and Cathie Yun.

## Footnote

PAT requires that the source of scarcity and an independent issuer be trusted not to collude. If they do collude, they can track users. This is unsuitable when "any party could play those two roles."
