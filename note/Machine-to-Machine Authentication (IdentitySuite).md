# Machine-to-Machine Authentication (IdentitySuite)

IdentitySuite's tutorial on the OAuth 2.0 Client Credentials flow: how services authenticate to each other when no user is present, walked through end-to-end in .NET with a token-caching client, Bearer attachment, and OpenIddict introspection on the receiving side.

---

## The argument in one paragraph

The article claims that service-to-service authentication is a solved, well-bounded problem with exactly one right default: the OAuth 2.0 Client Credentials flow, in which a service proves its own identity to an authorization server with a client ID and secret, receives a token that represents the *service* rather than any person, and presents it to the downstream service — with no login screen, no consent dialog, and no OpenID Connect layer, because there is no user identity to establish. If this is right, then most of the ceremony of user-facing OAuth is incidental rather than essential, and the interesting engineering questions reduce to plumbing: caching tokens, buffering expiry, and validating claims that look different (`sub` and scopes, no `name` or `email`) from what developers are used to. It is falsifiable in the sense that one could point to production systems where client-credentials-with-shared-secret is the wrong default — and the article itself never engages with those cases.

## Key quotes

> In M2M scenarios, there is no user. Service A is acting on its own behalf — not on behalf of any person.

The load-bearing distinction of the whole piece, and the reason the author says OIDC is unnecessary here: identity delegation is replaced by identity assertion, and everything downstream (no consent, no redirect, different claims) follows from it.

> The token it receives represents the service itself, not a human identity.

Worth pausing on because it quietly changes the security model: a service identity is shared by every instance and every request that service makes, so the token's granularity is coarser than any per-user token could be.

> Unlike user tokens, service tokens can and should be cached. Requesting a new token for every API call adds unnecessary latency and load on the authorization server.

Practical and correct, but stated without any of the caveats that make caching interesting — a long-lived cached bearer token in memory is precisely the asset the theft scenarios care about.

> If a user is involved at any point in the flow — even indirectly — the Authorization Code flow with PKCE is more appropriate. The Client Credentials flow is specifically for service identities, not user identities.

The article's one genuine decision rule, and a good one: the "even indirectly" clause is where most real-world misapplications of Client Credentials live (services acting *on behalf of* users should not be minting service tokens).

> `_tokenExpiry = DateTime.UtcNow.AddSeconds(expiresIn - 30); // 30s buffer`

The most honest line in the post is a code comment: the entire operational sophistication of the tutorial amounts to subtracting thirty seconds from an expiry, which is both admirably simple and a hint at how thin the guidance is.

## Critical analysis

What is non-obvious here is mostly the framing, not the mechanics. The observation that M2M is the case where you need *plain* OAuth 2.0 — no OIDC layer, because there is no user to authenticate — is a genuinely useful sorting rule that many developers get wrong by reflexively reaching for OpenID Connect everywhere. Likewise, the claim that `HttpContext.User` will contain a `sub` and scopes but no `name` or `email` prepares the reader for the disorienting experience of debugging a principal that describes a machine.

What is weak: this is a vendor tutorial optimised for a happy path. The client secret is stored in `appsettings.json` with no discussion of secret management; the token is a bare Bearer token with no mention of sender-constrained alternatives like DPoP or mTLS; introspection is recommended without noting its per-request cost or the token-vs-reference tradeoff; and there is no mention of token rotation, client secret rotation, or what happens when a service is compromised. The caching advice is directionally right but presented as if caching were free of security consequences.

What is left out is the entire modern argument about machine identity. Workload identity systems (SPIFFE/SPIRE, cloud-native workload identities, short-lived certificate-based identities) exist precisely because shared static client secrets scale poorly across hundreds of services — the article's model of "one client ID and secret per service" does not survive contact with a large microservices fleet. Nor is there any acknowledgment that the agent era is reopening these questions: when the "service" is an autonomous agent acting with delegated authority, the clean line between service identity and user identity that this article draws gets blurry fast.

Read alongside [[An Illustrated Guide to OAuth]], this is the companion case the visual guide covers lightly: the one flow with no human in it. Read alongside [[Your Access Token Was Stolen — Three Layers of Defence in Your Own OpenID Provider]], it is a study in what a vendor tutorial omits — that post assumes tokens get stolen and builds defences accordingly, while this one assumes they do not.

## Related

- [[An Illustrated Guide to OAuth]] — strengthens it by supplying the missing flow: Bhargava's walkthrough covers the user-delegation grants in detail, and this article is the natural complement showing what OAuth looks like when every human-mediated step is deleted.
- [[Your Access Token Was Stolen — Three Layers of Defence in Your Own OpenID Provider]] — complicates it sharply: Kozin's three layers (BFF, DPoP, fast revocation) are aimed at exactly the cached, long-lived bearer tokens this tutorial recommends producing, and none of those defences appear here.
- [[Authentication Is Largely Solved]] — nuances it: the article's confident "one grant type, four steps, done" tone supports the thesis that the mechanics are settled, while its silence on secret management and workload identity shows where the unsolved residue actually sits.
- [[You Dont Want Long-Lived Keys]] — complicates it from the other direction: Kuehn's argument that the best credential is one you configure once and never think about is the human-developer analogue of this article's cached service token, and the two positions expose how differently short-lived-vs-durable credentials get judged by audience.

---
*Sources: [[raw/machine-to-machine-authentication]], [[summary/machine-to-machine-authentication]]*
