---
url: https://www.alexedwards.net/blog/how-i-use-htmx-with-go
title: "How I Use HTMX with Go"
author: Alex Edwards
date_fetched: 2026-07-18
date_published: 2026-06-27
topics:
  - software-engineering-craft
---

# How I Use HTMX with Go — Summary

Alex Edwards' definitive field guide to integrating HTMX with Go web applications. The article walks through a complete working example, but the real value is in the patterns: a three-tier template architecture (base/pages/partials), the `htmlRenderer` abstraction that unifies full-page and partial rendering through a single `render()` method, `HX-Request` header detection for dual-mode endpoints that work for both HTMX interactions and direct URL visits, a redirect helper that handles the browser-intercept problem, error-handling configuration that swaps errors into the `<body>` instead of failing silently, and a set of opinionated HTMX defaults (cache disabled, inheritance disabled, indicator styles disabled) that prioritize predictability over convenience.

## Key patterns

1. **Template structure**: `base.tmpl` for layout, `pages/*.tmpl` for page-specific content, `partials/*.tmpl` for reusable fragments. Named templates with colon-namespacing (`page:title`, `partial:image:gopher`).

2. **htmlRenderer**: Clone the shared template set, append page-specific templates, execute the named template. Same method handles full pages and partials.

3. **Dual-mode handlers**: Check `HX-Request: true` header to decide whether to return a full page or a partial. Direct URL visits get the full page.

4. **Redirect management**: Use `HX-Redirect` header with 204 status for HTMX requests; fall back to standard 3xx for non-HTMX. Avoid `HX-Location` unless you're careful about route design.

5. **Error visibility**: Configure `responseHandling` so 4xx/5xx errors swap into `<body>` rather than failing silently in the console.

6. **HTMX defaults**: Disable localStorage cache, disable attribute inheritance, disable indicator styles, add a timeout. Err on the side of explicitness.
