# Machine-to-Machine Authentication

IdentitySuite's tutorial on the OAuth 2.0 Client Credentials flow argues that service-to-service authentication is a solved, almost boring problem — three steps, no user, no browser — and demonstrates it end-to-end in .NET with their own product as the authorization server.

---

## The argument in one paragraph

The article claims that when two backend services need to talk without a user, the Client Credentials flow is the complete answer: Service A authenticates to the authorization server with a client ID and secret, receives a token representing *the service itself* rather than any person, and presents it to Service B, which validates it via introspection. The falsifiable edge of the claim is the decision rule: if a user is involved at any point — even indirectly — Client Credentials is the wrong tool and Authorization Code with PKCE is the right one. This matters for this wiki because agents are the newest class of "service identity": non-human actors that call APIs on their own behalf, and the M2M pattern is the floor they stand on.

## Key quotes

> "This is machine-to-machine (M2M) authentication, and it is one of the most common scenarios in modern distributed systems."

Positioning M2M as ordinary infrastructure rather than a security specialty — which is fair; it is the most-used OAuth grant in practice, and the article treats it with correspondingly little ceremony.

> "The token it receives represents the service itself, not a human identity."

The load-bearing distinction of the whole piece. Everything else follows from it: no consent screen because there is no one to consent, no OIDC because there is no identity to establish, and `sub` names the client rather than a person.

> "Unlike user tokens, service tokens can and should be cached. Requesting a new token for every API call adds unnecessary latency and load on the authorization server. Cache the token and refresh it only when it is about to expire."

The one piece of operational advice in the article, and it is correct but incomplete — the sample caches in a singleton's memory, which is fine for one process and silently wrong for a fleet of instances that each hold their own copy of a long-lived secret.

> "If a user is involved at any point in the flow — even indirectly — the Authorization Code flow with PKCE is more appropriate. The Client Credentials flow is specifically for service identities, not user identities."

The article's only real opinion, and the most quotable thing in it. Delegation on behalf of a user is a categorically different problem from acting on one's own behalf, and blurring that line is where most auth bugs live.

## Critical analysis

What is non-obvious here is mostly the framing, not the mechanics. The observation that M2M is the case where you need *plain* OAuth 2.0 — no OIDC layer, because there is no user identity to establish — is a genuinely useful sorting rule that beginners often get backwards. And the closing decision rule, stated as a bright line between service identities and user identities, is the right mental model even though the article never explores the murky middle: agents acting *for* a user are neither pure service identity nor pure user identity, which is exactly the gap [[CSharp MCP Cross App Access]] and [[The Agent Access Model]] try to fill.

What is weak: this is vendor content wearing a tutorial's clothes. Every code sample routes through IdentitySuite, and the "no additional configuration required on the server side" claims are marketing. The technical content is also deliberately shallow in ways that matter. The client secret sits in `appsettings.json` in plaintext — the article never mentions secret storage, rotation, or the fact that a static shared secret is the weakest credential OAuth offers. There is nothing on mTLS or workload identity federation as stronger alternatives, nothing on token lifetime policy, nothing on what happens when a secret leaks (the answer, per [[Your Access Token Was Stolen — Three Layers of Defence in Your Own OpenID Provider]], is that a static client secret is a credential that never expires and cannot be made worthless after theft). The sample `TokenService` also has a subtle flaw it doesn't acknowledge: no locking, so concurrent first calls can race and fetch multiple tokens.

What is left out is the interesting part for this wiki. The article's world is services with human-assigned identities, registered by an admin in a UI. Agents break that model: they spawn at runtime, they need scoped, short-lived, auditable credentials that no admin registered in advance, and they act in a gray zone between "on their own behalf" and "on the user's behalf" that Client Credentials cannot express. The article is best read as the baseline the agent-identity work is reacting to — the pre-agent answer to a question agents have made urgent again.

## Related

- [[An Illustrated Guide to OAuth]] — Bhargava's explainer covers the user-delegation flows this article explicitly sets aside; read together they partition OAuth into its human and non-human halves, and this source supplies the half the guide's YNAB-and-Chase example never touches.
- [[Authenticating MCPs]] — Johnston's field guide treats MCP server auth as the live version of this problem; this source strengthens it by supplying the baseline Client Credentials pattern that MCP deployments extend when a user context has to ride along.
- [[Your Access Token Was Stolen — Three Layers of Defence in Your Own OpenID Provider]] — Kozin's three-layer defence (don't steal it, make it worthless, kill it fast) complicates this article's cheerful tutorial: a static client secret cached in memory fails all three layers, which the article never confronts.
- [[CSharp MCP Cross App Access]] — Sahasrabuddhe's two-hop token upgrade is the direct answer to the murky middle this article's bright-line rule cannot classify: an agent acting for an authenticated user is neither pure service identity nor pure user identity.

---
*Sources: [[raw/machine-to-machine-authentication]], [[summary/machine-to-machine-authentication]]*
