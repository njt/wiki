# ASP.NET Core Authentication Internals

Chris Klug's architectural deep-dive into ASP.NET Core's authentication system, based on reading the source rather than the docs. The system has been rewritten multiple times and emerged clean: everything builds on the four-method `IAuthenticationHandler` interface and a central `AuthenticationOptions` registry that acts as the source of truth for all schemes.

---

## Key Quotes

> "Simplicity does not precede complexity, but follows it."

Klug opened with this after discovering the authentication internals are actually clean, contrary to his talk's title. It's a design-philosophy thesis in one sentence: well-factored systems often look obvious only in retrospect, after the wrong abstractions have been tried and discarded. This applies far beyond authentication — it's a property of mature, repeatedly-rewritten subsystems.

> "The authentication system in ASP NET Core has been rewritten a couple of times. Apparently it's actually quite beautiful at this point. There are no ugly dirty details, there are no nooks and crannies."

The honesty of a speaker admitting their premise was wrong is refreshing. The rewriters earned their pay: the handler interface, the forwarding mechanism, and the options registry are genuinely well-factored.

> "The contents of the cookie is a capital N that's it."

The correlation cookie for CSRF protection in remote handlers stores the literal string `"N"`. The actual random value lives in the encrypted `AuthenticationProperties.Items` dictionary, which survives the round-trip as the OAuth `state` parameter. The cookie just proves possession — the `"N"` is essentially a sentinel. It's a clever two-channel verification pattern: the encrypted channel carries the secret, the cookie channel proves the browser made both requests.

> "This is not a thing that runs when you Sign in. This is a thing that runs on every incoming request and if you're unlucky, it runs on Every incoming CSS JavaScript image. Whatever request comes in, it will run this."

On `IClaimsTransformation.TransformAsync`. This is the kind of footgun that docs mention but don't scream about. A database call here becomes a per-request tax on your entire static asset pipeline.

> "Was this overly complicated to solve something that you can do with cookie authentication options? Yes. Was it fun? Yes."

After demonstrating the API-aware auth decorator that wraps the cookie handler via `IAuthenticationHandlerProvider` replacement. The self-awareness makes the point: the handler decoration pattern is genuinely useful even if *this specific use case* has a simpler solution.

## Key Themes

#dotnet #authentication #software-design #clean-architecture #middleware

### Authentication ≠ Sign-In

This is the load-bearing distinction. `IAuthenticationHandler` has four methods and none of them are sign-in or sign-out. Those live in separate interfaces (`IAuthenticationSignInHandler`, `IAuthenticationSignOutHandler`) because most schemes (OpenID Connect, OAuth, bearer tokens) only *prove* identity. Creating the session that remembers that identity across requests is cookie authentication's job. This is why `SignOutHandler` comes *before* `SignInHandler` in the interface hierarchy — remote providers need to redirect on sign-out to terminate the external session, but they don't create local sessions.

### The Handler Hierarchy as a Masterclass in Base-Class Design

The inheritance chain is unusually clean for a Microsoft framework:

1. **`IAuthenticationHandler`** — four methods, implement this if you want total control (and total responsibility)
2. **`AuthenticationHandler<TOptions>`** — adds forwarding logic, option resolution, time providers, events
3. **`AuthenticationHandler<TOptions>` + sign-out** — adds `HandleSignOutAsync`
4. **`AuthenticationHandler<TOptions>` + sign-in** — adds `HandleSignInAsync`
5. **`RemoteAuthenticationHandler<TOptions>`** — adds callback handling, correlation CSRF, backchannel HTTP, and the `IAuthenticationRequestHandler` interface for intercepting incoming requests

Each level adds exactly one concern. The base class does the forwarding check before calling the virtual/abstract method, so derived classes don't need to think about delegation. The `new` keyword on events properties is the only ugly bit.

### Forwarding Is the Hidden Superpower

Every authenticate/challenge/forbid call in the base class checks `ForwardAuthenticate`, `ForwardChallenge`, `ForwardForbid`, `ForwardDefault`, and the dynamic `ForwardDefaultSelector` func. Any of these being set means the handler delegates. This is how policy schemes work without being a special middleware component — they're just handlers that forward everything. The forwarding is baked into the base class, not the pipeline, which means it composes cleanly with all other handler behavior.

### The `AuthenticationProperties` Two-Channel Pattern

The `Items` dictionary is serialized, encrypted, base64-encoded, and survives round-trips (as the OAuth `state` parameter). The `Parameters` dictionary is `[JsonIgnore]` and only for in-process hand-offs. This clean separation — one channel for round-trip state, one for in-process communication — avoids the common mistake of leaking internal state into external-facing parameters. It's the same architectural instinct as OAuth's front-channel/back-channel distinction (see [[An Illustrated Guide to OAuth]]).

### Correlation ID CSRF: Simple and Effective

The `RemoteAuthenticationHandler` generates 32 random bytes, stores them in `Items[".xsrf"]`, issues a cookie with value `"N"`, and on callback validates both exist and match. The encrypted `Items` dictionary can't be tampered with by the client, so the correlation value is trustworthy. The cookie proves the same browser made both the outbound and inbound requests. It's a two-factor check for request continuity: what you sent (encrypted state) matches what you're holding (cookie).

### Claims Transformation Runs Everywhere

`IClaimsTransformation.TransformAsync` fires on every authenticated request — including static files. This is documented but easy to miss. The fix is either caching or moving transformation logic to sign-in events (which run once). Klug's demo showing the timestamp updating on every refresh makes the point viscerally.

### Handler Decoration via Provider Replacement

Replace `IAuthenticationHandlerProvider` in DI, intercept handler creation, wrap the original in a decorator implementing the same interfaces, return the decorator. This lets you modify behavior (return 401 instead of redirect for API calls) without touching the original handler's code. The pattern is general: any cross-cutting handler concern can be addressed through decoration rather than subclassing.

## The Demo Handlers

Klug built three demos, each overengineered for effect but genuinely instructive:

1. **Facial recognition handler** — embedded HTML pages served as assembly resources, in-memory session cache, full sign-in/sign-out implementation that doesn't use cookie auth
2. **Random identity provider** — fake remote auth handler that picks a random identity (Darth Vader, Master Yoda, Mario) to demonstrate the remote handler flow
3. **Policy scheme router** — routes to facial recognition or random identity based on User-Agent string (Edge gets random, Safari gets facial)

The demos are deliberately insecure (in-memory cache, no load balancing, no proper session management) and Klug explicitly warns against production use. But they're excellent teaching tools precisely because they don't hide complexity behind production hardening.

## Critical Analysis

**What's genuinely good:** The architecture is clean in the way only repeatedly-rewritten systems are. The four-method interface, the forwarding-as-base-class-behavior, the two-channel `AuthenticationProperties` pattern, and the handler decoration approach are all well-factored. The correlation ID CSRF pattern is elegant. This is a system where reading the source code is more illuminating than reading the documentation.

**What's underdeveloped in the talk:** Klug explicitly hand-waves the middleware pipeline ("whole other talk"), skips token storage/refresh/revocation entirely, doesn't cover handler testing strategies, and barely touches the interplay with authorization. The `AddScheme` limitation — overloads only work if you inherit from Microsoft's base classes — is mentioned but not solved. Multi-tenancy and dynamic scheme registration at runtime are not explored. Error handling and logging in production handlers are glossed over.

**The overengineering confession:** Klug calls himself "chief overengineering officer" and the demos lean into it. The API-aware auth decorator solves a problem that cookie authentication options already handle. But the *pattern* — handler decoration via provider replacement — is genuinely useful for cases that options don't cover. The embedded-resource HTML pages are, as he says, a "shit idea" — but they demonstrate that `IAuthenticationRequestHandler` can completely take over the pipeline.

**Relationship to [[Good API Design]]:** Klug's talk is an existence proof of Goedecke's thesis. The ASP.NET Core auth system succeeded because the team kept rewriting until the interface was boring-correct. The four-method `IAuthenticationHandler` is the kind of API that developers can hold in their heads — every method has a clear purpose, and the forwarding mechanism means you don't have to think about delegation unless you opt in. This is "boring on purpose" at the framework level.

**Relationship to [[Software Engineering Craft]]:** This talk is worth studying as an example of base-class design done right. The inheritance chain adds exactly one concern per level. The forwarding check happens before the virtual method call. The `AuthenticationProperties` pattern separates round-trip state from in-process state cleanly. These are craft-level decisions that compound — a thousand handlers benefit from the forwarding logic being in one place rather than reimplemented in each.

**Relationship to [[Flexible Authentication (Airbnb)]]:** Klug's `Challenge`/`ForwardChallenge` machinery is the framework-level ancestor of Airbnb's "Identify first, then Challenge" — both hand the *server* the decision of which challenge to present. The difference is altitude: ASP.NET gives you the plumbing to forward a challenge between schemes per request, while Airbnb builds the product layer (policy engine, Challenge Picker, server-driven screens) that decides *per person, per context* which challenge should lead. Airbnb's "the client never decides" is the product-scale expression of the forwarding instinct Klug reads in the source.

---
*Sources: [[raw/23c08df43340b155bc4a706d1aaaa937]], [[summary/23c08df43340b155bc4a706d1aaaa937]]*
*Last updated: 2026-08-07*
