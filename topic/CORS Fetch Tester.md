# CORS Fetch Tester

Simon Willison's browser-based debugging utility for understanding what CORS lets you see. A single-page tool that lets you send HTTP requests from the browser and inspect exactly which response headers and body data survive the CORS gauntlet — making visible the invisible filtering that normally confounds developers.

---

## Key Quotes

> "Send HTTP requests and inspect what the browser lets you see through CORS."

The tool's self-description is admirably precise. It doesn't promise to bypass CORS or show you everything — it shows you what the browser *lets* you see. This is a teaching tool disguised as a debugging utility: every blocked header is a lesson in web security architecture.

> "Only headers exposed by CORS are visible."

The most important sentence on the page. It explains *why* headers are missing without moralizing about it. Developers who've spent hours debugging "missing" response headers will recognize the pedagogical value of making this filtering explicit rather than silent.

> "If you omit the scheme, `https://` will be added."

A small UX decision that reveals Willison's instinct for friction reduction. The tool doesn't lecture you about URL correctness — it defaults to the secure option and moves on. This is the same polish that makes his other tools ([[Datasette]], `llm`, `sqlite-utils`) feel inevitable in retrospect.

## Key Themes

#tool #web #debugging #CORS #browser #simon-willison

## How It Works

A single-page web app. You construct an HTTP request (URL, method, headers, body), click "Send request," and the tool executes a browser `fetch()` call. The browser's built-in CORS enforcement does the rest — the tool simply surfaces what makes it through. Headers are filtered by `Access-Control-Expose-Headers`; bodies are visible or blocked depending on the server's CORS configuration.

The **Import from curl** feature is the killer detail: developers who already have a failing `curl` command can paste it directly into the tool and see what the browser sees, without manually translating headers and bodies between tools.

Request body support covers the three most common patterns: None, JSON, and URL-encoded forms. Response body rendering includes a **Format JSON** button — because CORS debugging often means staring at raw JSON API responses.

## Critical Analysis

This is peak Willison: a tool so minimal that most developers would dismiss it as "just a form around fetch()" — and that's exactly the point. The value isn't in the code (which is trivial) but in the *framing*. Browser DevTools already let you make fetch calls, but they don't tell you *why* you can't see certain headers. This tool makes CORS visibility the entire interface, turning a silent filtering layer into the thing you're explicitly inspecting.

The "Import from curl" feature is a masterclass in tool design. It accepts the input format developers already have (a curl command they're trying to debug) rather than forcing them to translate into a new interface. This is the same instinct behind `llm` accepting OpenAI-compatible APIs and `Datasette` speaking SQL: meet the user where they already are.

What's missing — and this is probably intentional — is any explanation of *how* to fix the CORS issues it surfaces. The tool is a diagnostic instrument, not a repair manual. You learn that your server isn't setting `Access-Control-Expose-Headers`, but you don't learn how to set it. This is the right call for a tool this focused; it's the CORS equivalent of a multimeter, not an electrician's apprenticeship.

The tool lives on `tools.simonwillison.net` alongside other single-purpose utilities — part of Willison's pattern of shipping tiny, polished tools that each do one thing well. In the era of AI-generated everything, there's something quietly radical about a hand-built HTML page that solves exactly one problem and solves it completely.

---

*Sources: [[summary/cors-fetch]]*
*Last updated: 2026-07-05*
