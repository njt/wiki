# HATEOAS — Hypermedia as the Engine of Application State

htmx.org's opinionated re-explanation of the most misunderstood constraint in REST: that the server, not the client, should drive application state by encoding all available actions directly in hypermedia responses. Using HTML rather than JSON to explain the concept, the essay makes the case that HATEOAS isn't an optional add-on to REST — it's the thing that makes REST REST, and JSON APIs that omit it are RPC, regardless of what their documentation says.

---

## Key Quotes

> "With HATEOAS, a client interacts with a network application whose application servers provide information dynamically through *hypermedia*. A REST client needs little to no prior knowledge about how to interact with an application or server beyond a generic understanding of hypermedia."

This is the essay's thesis compressed to a sentence. The server owns the state machine; the client is a universal renderer. The word "generic" is doing heavy lifting — a browser doesn't know what a bank account is, but it knows how to render `<a>` tags and submit `<form>` elements. That's the entire contract.

> "Only one link is available: to deposit more money. In the accounts current overdrawn state the other actions are not available, and this fact is reflected internally in *the hypermedia*."

The bank account example is the essay's most effective argument. The overdrawn-account response simply omits the withdraw/transfer/close links — the state transition is encoded as the *absence* of links, not as a `status` field the client must interpret. The browser doesn't know what "overdrawn" means; it just renders what it's given. This is HATEOAS in its purest form: the hypermedia *is* the application state.

> "It is this requirement of out-of-band information that distinguishes this JSON API from a RESTful API that implements HATEOAS."

The sharpest line in the essay. The distinction between REST and RPC isn't about HTTP verbs, status codes, or URL structure — it's about whether the client needs information that isn't in the response. If you need a Swagger doc to know what to do next, you're not doing REST.

> "I am getting frustrated by the number of people calling any HTTP-based interface a REST API. Today's example is the SocialSite REST API. That is RPC. It screams RPC. There is so much coupling on display that it should be given an X rating."

Roy Fielding, quoted in the essay. The frustration of watching your PhD thesis term get appropriated to mean "JSON over HTTP" is palpable. Fielding's point is that REST was a description of the web's architecture — HTML as the media type, links as the state transitions — and calling a JSON API "RESTful" is like calling a bicycle a "horseless carriage." It misses what the thing actually is.

> "This fact is strong evidence for the assertion that a natural hypermedia such as HTML is a practical necessity for building RESTful systems."

The essay's conclusion and its most controversial claim. The industry spent over a decade trying to bolt hypermedia controls onto JSON (HAL, JSON-LD, Siren, Collection+JSON, Uber) and uniformly failed. The essay argues this wasn't a failure of execution — it was a category error. JSON isn't a hypermedia format; HTML is. If you want HATEOAS, you need HTML (or something HTML-like). This is the theoretical foundation for why [[How I Use HTMX with Go|HTMX]] works: it lets you build applications that are genuinely RESTful because the server returns HTML, and HTML is the only hypermedia format with universal client support.

---

## Key Themes

#REST #HATEOAS #hypermedia #api-design #concept #htmx #JSON

- **Hypermedia as the state machine**: The essay's central claim is that in a RESTful system, the server *is* the state machine and the hypermedia responses *are* the state transitions. The client doesn't maintain a state machine that maps `status: "overdrawn"` to available actions — it simply renders whatever links the server returns. This inverts the typical JSON API architecture, where the client holds the state machine and the server is a dumb data store.

- **Out-of-band information as the diagnostic**: The essay provides a clean litmus test for RESTfulness: does the client need information that isn't in the response? If yes, it's not REST. This reframes the entire REST-vs-RPC debate from taxonomy (which is boring) to architecture (which matters). Out-of-band information is coupling; coupling is what REST was designed to eliminate.

- **HTML as the only production-grade hypermedia**: The essay's most opinionated claim is that HTML is effectively the only hypermedia format that works in practice. JSON hypermedia formats exist (HAL, Siren, JSON-LD) but none achieved significant adoption because JSON lacks the key hypermedia primitive: the `<form>` element. A `<form>` encodes the HTTP method, the target URL, the expected input fields, and the submission semantics in a single self-describing unit. A JSON `links` object encodes only the URL — everything else requires out-of-band knowledge.

- **The Richardson Maturity Model as a failed bridge**: The essay treats the Richardson Maturity Model (Level 0: HTTP as transport → Level 1: Resources → Level 2: HTTP verbs → Level 3: Hypermedia Controls) as an honest but unsuccessful attempt to retrofit REST onto non-hypermedia formats. The model correctly identified hypermedia controls as the distinguishing feature of REST, but the industry stopped at Level 2 and called it done. The essay's implicit argument is that Levels 0–2 without Level 3 aren't "immature REST" — they're a different architecture entirely.

- **Fielding's dissertation as descriptive, not prescriptive**: The essay notes that Fielding's dissertation described the early web architecture (HTML + HTTP) — it didn't invent a new one. HATEOAS wasn't a design goal; it was an observation about what made the web work. This matters because it means "REST without HATEOAS" is a contradiction in terms, not a simplification.

---

## Critical Analysis

**The essay is effective because it uses HTML to explain HATEOAS, where Wikipedia uses JSON.** This is more than a presentational choice — it's an argument. Wikipedia's JSON examples require the reader to imagine hypermedia controls bolted onto a format that doesn't natively support them. The HTML examples show hypermedia controls working as designed: `<a>` tags for links, `<form>` elements for mutations. The medium is the message.

**The overdrawn-account example is a microcosm of the entire architectural argument.** In the HTML version, the server stops returning the withdraw link. The client doesn't need to know why — it just stops showing the withdraw button. In the JSON version, the server returns `status: "overdrawn"` and the client must know what that means. The difference isn't aesthetic; it's about where the coupling lives. The HTML version couples the client only to the media type (HTML), which is stable. The JSON version couples the client to the specific API's semantics, which change whenever the business logic changes.

**The essay understates the practical barriers to HATEOAS adoption.** HTML hypermedia controls work beautifully for browser-to-server interactions where a human is in the loop — the human sees the links, clicks one, fills out the form, submits. But the dominant use case for JSON APIs is machine-to-machine communication, where there's no human to interpret the hypermedia. A machine client needs to know what a "deposit" link *means* to act on it programmatically. The essay acknowledges this implicitly (the browser "knows how to present hypermedia representations to a user") but doesn't address the machine-client case, which is arguably the one that matters for the API economy.

**The claim that "a natural hypermedia such as HTML is a practical necessity" is both the essay's strongest argument and its most debatable.** It's strong because the historical evidence supports it: no JSON hypermedia format achieved meaningful adoption. It's debatable because it conflates "hasn't happened yet" with "can't happen." The `<form>` element's superiority over JSON `links` objects is real, but it's not clear this is a fundamental limitation of JSON rather than a failure of imagination. A JSON equivalent of `<form>` — encoding method, URL, expected inputs, and validation rules in a structured format — is technically straightforward. The fact that nobody built one that caught on might be evidence that the industry didn't want HATEOAS, not that JSON couldn't support it.

**The essay's relationship to HTMX is the unstated thesis.** The essay appears on htmx.org for a reason. HTMX extends HTML's native hypermedia controls (adding `hx-get`, `hx-post`, `hx-target`, etc.) to enable richer interactions while keeping the HATEOAS architecture intact — the server still returns HTML, and the client still discovers actions from the response. The essay provides the theoretical foundation for why HTMX's approach is architecturally coherent while JavaScript SPAs backed by JSON APIs are not. In this reading, the essay isn't really about HATEOAS — it's about why HTMX exists. See [[How I Use HTMX with Go]] and [[Progressively Enhanced Forms with HTMX]] for the practical implementation patterns.

**Fielding's frustration is more interesting than the essay lets on.** "I am getting frustrated by the number of people calling any HTTP-based interface a REST API" is a quote from 2008. The industry heard this complaint and ignored it for nearly two decades. The essay treats this as evidence that the industry was wrong, but there's another reading: the industry collectively decided that "REST" meant "JSON over HTTP with pretty URLs" and that genuine HATEOAS wasn't worth the cost. The term was captured, and no amount of blog posts from its originator could recapture it. This is a case study in how technical terms evolve in ways their coiners can't control — the same dynamic that gave us "serverless" servers, "edge" functions that run in datacenters, and "AI" that's really statistical pattern matching.

**The essay's binary framing (REST = HATEOAS, everything else = RPC) is rhetorically effective but architecturally crude.** There's a spectrum between "the client hardcodes every URL and status code" and "the client is a pure hypermedia renderer." Most real APIs sit somewhere in the middle: the client knows the entry-point URL and the media type, discovers resource URLs from responses, but hardcodes knowledge of what specific link relations mean. The essay's absolutism serves its argument but doesn't help someone trying to make a practical API design decision. [[HTTP API Design Guide (Heroku)]] and [[Good API Design]] offer the pragmatic middle ground this essay deliberately avoids.

---

*Sources: [[raw/hateoas]], [[summary/hateoas]]*
*Last updated: 2026-08-08*
