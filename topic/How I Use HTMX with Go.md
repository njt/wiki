# How I Use HTMX with Go

Alex Edwards' practical field guide to integrating HTMX with Go web applications — the most detailed public reference for the server-side HTML rendering pattern that the GUS Stack endorses. It covers template architecture, the `htmlRenderer` abstraction, dual-mode endpoints, redirect management, error handling, and opinionated HTMX configuration defaults.

---

## Key Quotes

> "When I want to add sprinkles of interactivity to a web application, I'm a big fan of using HTMX. I like that it makes it easy to give interactions a smooth app-like feel, I like that it minimizes the amount of JavaScript that I have to write, and I like that it allows me to keep the consistency and safety of server-side HTML rendering with Go's html/template package."

The thesis compressed to a sentence. Edwards isn't arguing HTMX is universally correct — he's arguing it's correct *for the class of applications where server-side rendering already makes sense*. The word "sprinkles" is doing real work here: HTMX is for enhancing server-rendered pages, not for building SPAs. The consistency argument — that Go's `html/template` package provides type-safe, context-aware escaping — is the one most HTMX advocacy misses. You're not just avoiding JavaScript; you're staying inside a safety perimeter that the Go toolchain enforces at compile time.

> "Because we've set up our htmlRenderer type so that the shared template set already includes all partials, it's sufficient for us to call render() like this without passing in any additional file paths."

The `htmlRenderer` is the article's real contribution. It's a small abstraction — maybe 30 lines of Go — but it solves the core problem of HTMX+Go integration elegantly: the same function renders both full pages (execute `"base"`) and partials (execute a named fragment). The shared template set is cloned per-request, so there's no concurrency issue. Page-specific templates are parsed on demand. The entire API surface is `render(w, status, data, templateName, additionalFiles...)`. This is the kind of abstraction that seems obvious in retrospect but takes years of iterating on real applications to crystallize.

> "If the request is not coming from HTMX, we should return a full HTML page that contains the matching user details."

The dual-mode handler pattern — check `HX-Request: true`, return a partial if present, return a full page if absent — is the article's most practically useful technique. It means every HTMX endpoint is also a shareable URL. Someone can bookmark `/users/search?query=leo` and get a proper HTML page, not a raw `<tr>` fragment. This is progressive enhancement done right: the HTMX version is strictly better (no full-page reload), but the non-HTMX version works. The `Vary: HX-Request` header on all responses is a small cost for cache-correctness that Edwards correctly argues is worth paying everywhere rather than remembering to set it per-endpoint.

> "I tend to disable the HTMX cache completely by setting historyCacheSize to 0. Caching pages in local storage is a source of bugs and security issues, so I think it's simpler and better to just disable it completely."

Edwards' configuration defaults are opinionated in the best way: every choice has a stated reason, and several of them anticipate future HTMX defaults. Disabling the localStorage cache, disabling attribute inheritance, and disabling indicator styles all follow the same principle — *explicit over implicit*. When an agent or a teammate encounters HTMX attributes in your codebase, they shouldn't have to also know about invisible inheritance rules or cached state that might alter behavior. This is [[Guardrails and Feedback Loops]] applied to frontend architecture: make the system's behavior legible from its source.

## Key Themes

#go #htmx #web-development #template-architecture #pattern #server-side-rendering

- **The htmlRenderer as the missing Go+HTMX primitive**: Edwards doesn't name it as a pattern, but the `htmlRenderer` type is the kernel that makes everything else work. It solves three problems at once: template caching (shared set parsed once at startup), request-scoped isolation (clone per request), and a unified API for full pages and partials. This is the Go ethos in microcosm — a small, composable type that does one thing well, rather than a framework that does everything opaquely.

- **Dual-mode endpoints as progressive enhancement**: The `isHTMXRequest(r)` check enables the same URL to serve both an HTMX partial and a full HTML page. This is the architectural move that prevents HTMX from creating a second-class web that only works with JavaScript. It's also the pattern that makes [[The GUS Stack — Go, Unix, SQLite]] viable — without dual-mode handlers, HTMX endpoints create URLs that are broken when shared. Edwards' approach makes every interaction a first-class URL.

- **Template architecture as namespace design**: The colon-namespaced template names (`page:title`, `page:content`, `partial:image:gopher`) are a convention that scales. In a small app, flat names work fine. In a larger app with dozens of pages and partials, the namespace prevents collisions and makes the call chain traceable. The decision to keep page-specific fragments in the same file as the page (rather than forcing everything into `partials/`) is pragmatic — colocation beats purity when a fragment is only used in one place.

- **Configuration as documentation**: Edwards' HTMX config block reads like a design document. Every setting has a justification: `historyCacheSize: 0` because localStorage caching is a bug factory, `disableInheritance: true` because explicit attributes are easier to debug, `includeIndicatorStyles: false` because CSS should live with your other CSS. The meta tag isn't just configuration — it's a statement of architectural intent that a teammate or agent can read and understand.

- **The redirect problem as a case study in leaky abstractions**: The `HX-Redirect` vs `HX-Location` discussion is the article's most honest moment. Edwards walks through why `HX-Location` (which avoids a full-page reload) seems better but creates an unsolvable ambiguity — the target handler can't tell if the request is from an `HX-Location` redirect or a normal HTMX interaction. The conclusion (use `HX-Redirect`, accept the full-page reload) is a case study in choosing simplicity over cleverness. This is the same judgment that leads to the GUS Stack's preference for HTMX over TypeScript: take the hit on interactivity to avoid the complexity cliff.

## Critical Analysis

**This is the implementation manual the GUS Stack doesn't provide.** Zoschke's [[The GUS Stack — Go, Unix, SQLite]] says "use HTMX as much as possible" but doesn't explain how. Edwards' article fills that gap with production-grade patterns. The two articles should be read together: Zoschke provides the *why* (training-data density, boring technology, agent compatibility), and Edwards provides the *how* (template structure, renderer abstraction, dual-mode handlers, config defaults). Together they form the most complete public reference for building Go+HTMX applications.

**The `htmlRenderer` should be an open-source library.** It's ~50 lines of Go, well-scoped, and solves a problem every Go+HTMX project hits. Edwards keeps it inline as a tutorial pattern, but it's mature enough to extract. The fact that it hasn't been extracted tells you something about the Go+HTMX ecosystem: it's still a collection of patterns rather than a framework, and the community seems to prefer it that way. Whether that preference survives agentic development — where agents do better with framework conventions than bespoke patterns — is an open question.

**The article's scope is a feature, not a bug.** Edwards deliberately omits middleware, auth, testing, database access, and deployment. This isn't an oversight — it's the article's thesis expressed structurally. HTMX integration is a thin layer, not a framework. The patterns here compose with whatever middleware stack, auth system, and database you already have. This is the opposite of the Rails/Hotwire approach, where the frontend library assumes the rest of the stack. Go+HTMX is a la carte; Edwards shows you how to order the HTMX dish without prescribing the rest of the meal.

**The back-button discussion reveals HTMX's fundamental tension.** HTMX wants to make AJAX interactions feel like native page navigation, but the browser's history API wasn't designed for partial page updates. The `historyRestoreAsHxRequest` setting, the localStorage cache, and the `HX-Current-URL` header are all patches over the impedance mismatch between HTMX's model (swap HTML fragments into the DOM) and the browser's model (navigate between complete pages). Edwards navigates this carefully, but the complexity is real. Every HTMX project eventually discovers that "it's just HTML" is true at the markup level but false at the architecture level — you're still building a client-side application, just one where the client-side logic ships in a library rather than your own JavaScript.

**The absence of a testing strategy is the biggest gap.** Edwards shows how to build the thing but not how to verify it stays built. The GUS Stack article recommends `rod` + `rodney` for headless Chrome DOM verification. Edwards doesn't mention testing at all. For an article this thorough about production patterns, the omission is conspicuous. The implicit assumption is that HTMX endpoints are testable the same way as any HTTP handler — and they mostly are — but HTMX introduces failure modes (cache-miss back-button navigation, `HX-Redirect` header handling, error-response swapping) that unit tests of handlers alone won't catch. [[A New Era for Software Testing]] and [[Agentic Testing]] are relevant here: the complement to Edwards' patterns is a testing strategy that exercises the full HTMX lifecycle, not just the Go handler in isolation.

**The article is a snapshot of mid-2026 Go web development taste.** `embed.FS` is standard (since Go 1.16). The `html/template` package is standard library. The pattern of parsing shared templates once and cloning per request is established Go practice. What's new is the HTMX-specific glue — and Edwards' version of it is clean enough that it will likely become the reference pattern. The specific HTMX version (2.0.10) and configuration defaults match what the HTMX project itself is moving toward (cache disabled, inheritance disabled). The article will age well because it's built on stable primitives.

---

*Sources: [[raw/how-i-use-htmx-with-go]]*
*Last updated: 2026-07-18*
