# PACT: Anonymous Credentials for the Web

Mozilla's proposal for solving the web's bot problem without destroying privacy: replace identity checks with cryptographic rate-limiting credentials. Instead of proving *who you are*, you prove you're subject to a rate limit — using Privacy Pass tokens, zero-knowledge issuer blinding, and stateful credentials that sites can adjust without being able to track. The architecture sketches an open ecosystem where any site can vouch for scarcity (subscriptions, phone numbers, account standing) and the cryptography ensures "no more than the minimum information gets through: a single bit."

---

## Key Quotes

> "The more effectively they protect their privacy, the harder it is for websites to distinguish them from bots and the worse the treatment they receive."

This is the trap PACT tries to break. Every privacy protection — VPNs, private windows, anti-fingerprinting — makes you look more like a bot. The result is a race to the bottom where privacy-respecting users get punished, and the only way to prove humanity is to surrender identity. PACT inverts this: prove scarcity, not identity.

> "Bots' harms arise from their ability to operate beyond human scale."

The article's core reframing. Sites don't actually need to know who you are — they need to know you can't send 10,000 requests per second. This is the insight that makes the whole architecture possible: rate limiting as the real requirement, identity as an overfit solution.

> "Both of these approaches are ultimately hostile to users and to the openness of the web."

On Google's abandoned Web Environment Integrity and Apple's Private Access Tokens. Google's version would have made new browsers nearly impossible if sites adopted allow-lists. Apple's ties access to expensive hardware from a small vendor set. The open-source, multi-anchor approach PACT proposes is explicitly a reaction to both.

> "The system degrades to today's experience rather than locking the user out."

Crucial design choice. If you don't have any Endorsements from trusted Anchors, you still get a Credential — you just go through existing mechanisms (CAPTCHAs, logins). No new gates, no new lockouts. This is how you get adoption: make the happy path better without making the fallback worse.

> "The exact value is never leaked to the site."

On Anonymous Credit Tokens — the stateful credential primitive. A site can verify your counter exceeds a threshold, and can increment or decrement it based on behavior, but never learns the actual value. This is genuinely clever: behavioral rate limiting without behavioral tracking.

---

## Key Themes

- **#concept** — Rate limiting over identity. The fundamental reframe: bots harm through scale, not through anonymity. Solve scale, preserve anonymity.
- **#concept** — Scarcity anchoring. Rate limits need something an attacker can't replicate cheaply. Hardware is one option (Apple's approach), but phone numbers, email addresses, subscriptions, and account standing are open alternatives that don't tie the web to device manufacturers.
- **#concept** — Issuer blinding. The hardest technical problem in the architecture: proving you hold a token from *some* trusted Anchor without revealing *which one*. The privacy of the whole system depends on this. They acknowledge generic ZK approaches are too slow and "bespoke approaches" are needed — which is honest about the gap between concept and deployment.
- **#concept** — Uniform initial access. The dirty trade-off they're upfront about: because the Moderator can't see which Anchor you used (privacy requires this), everyone starts at the same access level — the level of the weakest Anchor in the pool. Access accrues through good behavior. This is both elegant and potentially unfair to users of strong Anchors.
- **#tool** — Privacy Pass, the IETF-standardized unlinkable token protocol originally built for Cloudflare + Tor CAPTCHAs in 2018. Already deployed by Apple, Chrome, and Kagi.
- **#tool** — Anonymous Credit Tokens (ACT), the stateful credential primitive that enables rate-limit counters invisible to the verifier. This is the novel crypto that makes PACT more than just Privacy Pass with extra steps.
- **#tool** — Prio, Mozilla's multiparty computation system for private aggregate scoring. Used here so sites can evaluate Anchor quality without learning which individual users used which Anchor.
- **#pattern** — The three-role architecture: Anchor (issues scarcity endorsements), Moderator (rate-limits), Client (carries credentials). Clean separation of concerns. Anchors don't need to know where you browse; Moderators don't need to know who vouched for you.

---

## Critical Analysis

**The good:** This is the most thoughtfully designed proposal for web-scale anonymous anti-abuse I've seen. It correctly diagnoses the problem (rate limiting, not identity), correctly critiques the incumbents (Google's attestation dystopia, Apple's hardware lock-in), and correctly identifies the hard cryptographic problems (issuer blinding, stateful credentials, aggregate accountability). The "degrades to today's experience" fallback is smart product thinking disguised as architecture. The explicit welcome to AI agent traffic — treating agents as first-class citizens of the credential ecosystem rather than as a threat to be detected — is forward-looking in a way most anti-abuse proposals aren't.

**The hard parts they're honest about:** Issuer blinding performance. The uniform-initial-access trade-off (your VPN subscription endorsement gets you the same starting rate limit as someone who verified a throwaway email). The need for bespoke cryptography rather than off-the-shelf ZK. The early stage — "has the right shape, but many of the details still need to be worked out."

**What they don't address:** The bootstrapping problem. Who are the first Anchors? If Mozilla, Cloudflare, and Google are the initial set, how is that meaningfully different from the "small, hard to change set of vendors" they criticize Apple for? The article gestures at an open ecosystem but the path from three collaborators to hundreds of Anchors is unclear. Also: what stops a malicious Anchor from endorsing bots at scale? The aggregate scoring via Prio helps detect this after the fact, but the damage window between "Anchor goes rogue" and "Anchor gets detected and removed" could be significant.

**The strategic play:** This is Mozilla doing what Mozilla should do — proposing open web standards that preserve privacy while solving real problems. The collaboration with Cloudflare and Chrome gives it credibility. The IETF/W3C standardization path is the right one. But Mozilla's browser market share is single digits, and web standards live or die on implementation. Chrome's participation is encouraging; Apple's absence from the collaborator list is notable given their existing Privacy Pass deployment.

**The bottom line:** PACT is a genuinely good idea at the architecture level, with hard cryptographic problems still unsolved. If the bespoke issuer blinding and ACT primitives work at production scale, this could replace the creeping identity-demand trend with something that preserves both privacy and abuse protection. If they don't, it joins the graveyard of elegant privacy protocols that couldn't clear the performance bar. Worth watching either way — the diagnosis alone (rate limiting over identity, scarcity over attestation) is useful for thinking about any system that needs to distinguish humans from bots.

---

*Sources: [[raw/pact-anonymous-credentials]]*
*Source URL: https://hacks.mozilla.org/2026/06/pact-anonymous-credentials-for-the-web/*
*Author: Dennis Jackson, Mozilla*
*Published: 2026-06-23 | Fetched: 2026-06-24*
