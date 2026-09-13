# Your Access Token Was Stolen — Three Layers of Defence in Your Own OpenID Provider

Rinat Kozin's engineering writeup of redb.Identity, an Apache-2.0 OAuth 2.1/OIDC provider on .NET, organised around one uncomfortable question: what actually happens when an access token is stolen? The answer is three layers — don't let it be stolen (BFF), make the stolen thing worthless (DPoP), kill it fast (revocation) — followed by the worst case, a stolen database, where the interesting answers are Argon2id, a pepper that never touches the database, and a key ring that refuses to boot unprotected.

---

## Key quotes

> "An access token is a bearer token by default, and that phrase means exactly what it says: whoever holds it, is you."

The cleanest one-line statement of the bearer-token problem. The four theft vectors that follow (own-code XSS, a compromised npm dependency, browser extensions, localStorage) share one property: the attacker walks away with the token itself, needing neither the victim's browser, network, nor session.

> "A stolen token is used from someone else's machine, which is precisely the signal impossible-travel detection exists for. XSS inside the victim's browser arrives from their own address... So on detectability the BFF loses."

The honest trade-off table. BFF wins four of five rows — attacker location, duration, tab-close survival, replayability — and loses the fifth. Activity inside the victim's browser comes from the victim's IP, user-agent, and working hours, behaviourally almost indistinguishable from the person. And the caveat under the caveat: they don't have anomaly detection yet, so the row they lose is one they can't even cash in on.

> "You took a cookie, so bring CSRF protection, no matter how good the other flags are."

The BFF doesn't remove risk, it changes its shape: token theft becomes CSRF. SameSite=Lax is not a substitute for an antiforgery token — it fails against your own subdomain and stops helping the moment a flow needs SameSite=None. Twelve state-changing forms, every one carries a token.

> "The key chain ends at a root that is not in the dump."

The database-theft section's core. Argon2id at 64 MiB (24 GB of video memory holds ~380 parallel cracking lanes where bcrypt's 4 KB footprint allows tens of thousands); legacy bcrypt hashes migrated by upgrade-on-login rather than a forced reset; recovery codes hashed with a pepper supplied by environment variable, never a table; signing keys encrypted under DataProtection; and the key ring encrypted at rest with a fail-to-start condition in production. The honest counterweight: a dump still yields PII, org structure, session metadata, and the audit trail — a reportable personal-data breach with zero passwords cracked. And if the backup ships with the config file holding the root key, "the protection collapses to zero."

> "Everything above can be asserted about any server, and usually is."

So they ran the official OpenID Foundation conformance suite as an external arbiter — and it found a real bug none of their own tests caught: scope-derived claims (phone, address) landing in the id_token, which is forwarded to third parties and written to logs as proof of sign-in. They don't claim OpenID Certified™ (a paid trademark); only that the suite ran and these are the results.

## Key themes

- #concept — the bearer token as a default worth unmaking; proof-of-possession (DPoP, RFC 9449) as the alternative that works where a BFF can't
- #pattern — BFF as "nothing there to attack by construction"; pull-with-cursor revocation as change-data-capture applied to session state; upgrade-on-login password migration; fail-to-start as an encryption enforcement mechanism
- #tool — redb.Identity itself, and the OpenID Foundation conformance suite as an external arbiter against self-confirming tests

## Analysis

The three layers map cleanly onto prevention, containment, and recovery — but the standout architectural idea is the revocation feed. Classic OIDC backchannel logout is push: the provider knocks on every registered application, and the first partition or crashed replica silently misses the knock. The pull feed of revoked session IDs with a cursor turns "sign out everywhere" into an eventual-consistency catch-up problem — the same shape as change data capture, applied to session state — and the BFF checks the list on every request via `OnValidatePrincipal`, so revocation kills the live UI session across every replica, not just the one that received the logout call. That is an idea more identity systems should steal.

Equally notable is the admission density. A table scoring the chosen architecture against the SPA alternative including the row it loses; a "what is not there yet" section listing risk-based authentication, step-up (RFC 9470), trusted devices as first-class entities, SAML 2.0, Rich Authorization Requests, and RFC 8705 mTLS — with the mTLS distinction drawn precisely (transport-level client certificates vs. mTLS as OAuth client authentication plus `cnf.x5t#S256` token binding); and a refusal to claim FAPI 2.0 because "a profile is not a set of checkboxes, it is passing the corresponding conformance plan."

Scepticism is still warranted. This is, at bottom, a product announcement ("a ⭐ on GitHub helps others find it"); the conformance run is self-executed rather than certified; and every security claim is about a reference implementation nobody else has audited. The BFF detectability table leans on anomaly detection the author admits doesn't exist yet — the fifth row is a promissory note. The mobile story (system browser, PKCE, OS key store, DPoP) assumes secure storage actually holds, which is an assumption rather than a demonstration.

Even so, as a worked example of taking "what if the token gets stolen" seriously — and of writing down the limits in the same breath as the claims — this is a model. The takeaway that generalises to any server takes five minutes to check: what hashes your passwords and whether upgrade-on-login exists, and where the key that encrypts your private signing keys lives. "If it is in the same database, the encryption is decorative, and the worst case of a stolen dump is wide open."

## Related pages

- [[An Illustrated Guide to OAuth]] — Bhargava's visual explainer ends where this source begins: it shows why each OAuth mechanism closes an attack vector, while this piece picks up after issuance and asks what happens when the issued token leaks anyway.
- [[Authentication Is Largely Solved]] — complicates Windley's thesis: authentication may be "about as good as we can reasonably ask," but everything this source describes — bearer-token theft, CSRF, key management, revocation propagation — lives in the post-authentication session layer that is decidedly not solved.
- [[ASP.NET Core Authentication Internals]] — same stack, complementary altitude: Klug tours the `IAuthenticationHandler` machinery from the source, while Kozin shows that machinery deployed in anger — cookie auth, DataProtection, antiforgery — inside a working BFF.
- [[The Agent Access Model]] — nuances Cloudflare's task-scoped credentials with a concrete mechanism: DPoP's proof-of-possession makes a token non-transferable in exactly the way agent credential theft (a hijacked harness walking off with a bearer token) demands, and the pull-based revocation feed answers how to un-grant at machine speed.

---
*Sources: [[raw/your-access-token-was-stolen-now-what-three-layers-of-defence-in-your-own-openid-provider-4hon]], [[summary/your-access-token-was-stolen-now-what-three-layers-of-defence-in-your-own-openid-provider-4hon]]*
*Last updated: 2026-09-13*
