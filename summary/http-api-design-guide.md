---
url: https://geemus.gitbooks.io/http-api-design/content/en/
title: "HTTP API Design Guide"
author: Wesley Beary (geemus) and the Heroku Platform API team
date_fetched: 2026-07-08
date_published: 2013
---

A set of HTTP+JSON API design conventions extracted from the team that built
the Heroku Platform API. The guide codifies patterns for consistency across
Heroku's internal APIs and aims to let teams focus on business logic rather
than design debates.

Key positions: require TLS everywhere with no exceptions — reject or 403
non-TLS requests rather than redirecting. Require API versioning in the
`Accept` header from day one; never use a default version. Use `ETag` headers
for caching, `Request-Id` (UUID) for traceability, and `Range` headers to
paginate large responses.

On request design: accept serialized JSON bodies, use plural downcased
dash-separated resource names, prefix special actions with `actions/`, minimize
path nesting, and support both IDs and human-readable names for resource
lookup. On response design: return the correct HTTP status code, include full
resource representations wherever possible, use UUIDs for IDs, ISO8601 UTC for
timestamps, nest foreign-key relations as objects, and return structured error
bodies with machine-readable `id`, human-readable `message`, and an optional
help `url`.

Tooling recommendations include prmd for machine-readable JSON schema
management and Markdown doc generation, plus executable terminal examples so
users can try the API with zero friction. The guide also covers rate-limiting
via token bucket with `RateLimit-Remaining` headers and a stability
classification system (prototype/development/production) for communicating
endpoint maturity.
