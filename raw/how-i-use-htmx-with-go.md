---
url: https://www.alexedwards.net/blog/how-i-use-htmx-with-go
title: "How I Use HTMX with Go"
author: Alex Edwards
date_fetched: 2026-07-18
date_published: 2026-06-27
---

Not sure how to structure your Go web application? My new book guides you through the start-to-finish build of a real world web application in Go — covering topics like how to structure your code, manage dependencies, create dynamic database-driven pages, and how to authenticate and authorize users securely. Mid-year sale: 30% off until the end of July! Take a look! When I want to add sprinkles of interactivity to a web application, I'm a big fan of using HTMX. I like that it makes it easy to give interactions a smooth app-like feel, I like that it minimizes the amount of JavaScript that I have to write, and I like that it allows me to keep the consistency and safety of server-side HTML rendering with Go's html/template package.

In this post I'm going to run through how I typically use HTMX in conjunction with Go. Although I'm going to talk a bit about how HTMX works, the main focus is going to be on the Go side of things. Specifically:

- The patterns I use for structuring HTML templates and sending back partial and full-page HTML responses to HTMX
- Managing redirects and errors when using HTMX
- The standard HTMX configuration settings that I use, and why

To illustrate these things, we'll run through the build of a small application that ultimately implements a filter on a list of users.

## Project setup

```
$ go mod init example.com/htmx
$ mkdir -p assets/static/css assets/static/img assets/static/js assets/html/partials assets/html/pages cmd/web
$ touch assets/efs.go assets/html/base.tmpl assets/html/partials/images.tmpl assets/html/pages/home.tmpl cmd/web/main.go cmd/web/handlers.go cmd/web/html.go
```

## Installing HTMX

The author downloads HTMX locally rather than using a CDN, along with Bamboo CSS and a gopher image.

## The HTML templates

The template structure follows a base/pages/partials pattern:

```
assets/html
├── base.tmpl
├── pages
│   └── home.tmpl
└── partials
    └── images.tmpl
```

- `assets/html/base.tmpl` — common HTML 'layout' markup for all web pages
- `assets/html/pages/` — page-specific content for individual web pages
- `assets/html/partials/` — reusable chunks of HTML markup

The base template uses Go's `{{define "base"}}...{{end}}` with named template actions like `{{template "page:title" .}}` and `{{template "page:content" .}}`. The colon character is used as a namespace separator.

HTMX is imported with the `defer` attribute so it's fetched in parallel but executed after the DOM is built.

## Embedding the assets

Since Go 1.16, the author embeds HTML files and static assets using `//go:embed`:

```go
//go:embed "html" "static"
var files embed.FS

var (
    HTMLFiles   = sub(files, "html")
    StaticFiles = sub(files, "static")
)
```

This creates two sub-filesystems — `HTMLFiles` and `StaticFiles` — rooted in their respective directories, providing clear separation and eliminating path prefixes.

## HTML template rendering — the htmlRenderer pattern

The core contribution of the article is the `htmlRenderer` type:

1. Parses a set of shared templates at startup (base + all partials)
2. Has a `render()` method that clones the shared template set, optionally parses additional templates, executes a named template, and writes the HTTP response

```go
type htmlRenderer struct {
    templateFS      fs.FS
    sharedTemplates *template.Template
}

func newHTMLRenderer(templateFS fs.FS, sharedTemplateFiles ...string) (*htmlRenderer, error) { ... }
func (h *htmlRenderer) render(w http.ResponseWriter, status int, data any, templateName string, additionalTemplateFiles ...string) error { ... }
```

The `render()` method can send either complete HTML pages (by executing the `"base"` template) or specific partials (by executing a named partial template) — using the same function for both.

## Rendering partials

In the handler, rendering a partial is as simple as:

```go
err := app.html.render(w, http.StatusOK, width, "partial:image:gopher")
```

Because all partials are already in the shared template set, no additional file paths are needed.

## Checking if a request is from HTMX

Requests from HTMX include an `HX-Request: true` header. The author creates a helper:

```go
func isHTMXRequest(r *http.Request) bool {
    return r.Header.Get("HX-Request") == "true"
}
```

This allows handlers to return either a full page or a partial based on the request source, so direct URL visits (e.g., sharing a search link) get a full HTML page instead of a raw partial.

The `Vary: HX-Request` header is set on all responses to tell caches that responses may differ based on this header.

## Managing redirects

Browsers automatically follow 3xx responses before HTMX can see them. The solution is the `HX-Redirect` header with a 2xx response:

```go
func redirect(w http.ResponseWriter, r *http.Request, url string, code int) {
    if isHTMXRequest(r) {
        w.Header().Set("HX-Redirect", url)
        w.WriteHeader(http.StatusNoContent)
        return
    }
    http.Redirect(w, r, url, code)
}
```

`HX-Redirect` triggers a full-page reload. The alternative `HX-Location` avoids the reload but has a problem: the handler can't distinguish between an HTMX request following an `HX-Location` redirect (should return full page) and a normal HTMX request (should return partial).

## Managing errors

By default, HTMX doesn't swap in 4xx/5xx responses — it logs errors to the console. The author configures `responseHandling` to change this:

- 204 No Content → no DOM changes
- 422 Unprocessable Content → swap into target as normal (for form validation errors)
- All other 4xx/5xx → swap into `<body>` element (full-page error display)
- All other responses → swap into target as normal

## Back-button behavior

HTMX caches pages in localStorage for history navigation. If there's a cache miss, HTMX re-fetches — but with `HX-Request: true`, which would return a partial. The fix: `historyRestoreAsHxRequest: false`.

## Additional HTMX configuration

The author's recommended defaults:

```json
{
    "includeIndicatorStyles": false,
    "historyCacheSize": 0,
    "historyRestoreAsHxRequest": false,
    "responseHandling": [
        {"code": "204", "swap": false},
        {"code": "422", "swap": true},
        {"code": "[45]..", "swap": true, "target": "body"},
        {"code": "...", "swap": true}
    ],
    "timeout": 5000
}
```

- `includeIndicatorStyles: false` — define indicator styles alongside other CSS
- `historyCacheSize: 0` — disable localStorage caching entirely (simpler, fewer bugs; will be the default in future HTMX versions)
- `disableInheritance: true` — prefer explicit HTMX attribute declarations
- `timeout: 5000` — project-specific default timeout

## Page-specific layouts

For larger applications, a "layout" template can sit between the base template and page content:

```go
func (app *application) adminOrders(w http.ResponseWriter, r *http.Request) {
    err := app.html.render(w, 200, nil, "base", "layouts/admin.tmpl", "pages/admin-orders.tmpl")
    ...
}
```

## Current browser URL

HTMX sends the user's actual browser URL in the `HX-Current-URL` header, which may differ from `r.URL` in Go handlers. A helper parses it:

```go
func browserURL(r *http.Request) (*url.URL, error) {
    cu := r.Header.Get("HX-Current-URL")
    if cu != "" {
        return url.Parse(cu)
    }
    return r.URL, nil
}
```
