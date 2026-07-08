---
url: https://geemus.gitbooks.io/http-api-design/content/en/
title: HTTP API Design Guide
author: Wesley Beary (geemus) and the Heroku Platform API team
date_fetched: 2026-07-08
date_published: 2013
---

# HTTP API Design Guide

A set of HTTP+JSON API design practices, extracted from work on the Heroku Platform API. The guide informs additions to that API and also guides new internal APIs at Heroku. It aims for consistency and focusing on business logic while avoiding design bikeshedding. It assumes familiarity with HTTP+JSON API basics.

Source: https://github.com/interagent/http-api-design

---

## Foundations

### Separate Concerns

Keep things simple while designing by separating the concerns between the various parts of the request and response cycle. Following straightforward rules in this area frees up attention for more complex challenges.

Requests and responses target a particular resource or collection. The path should indicate identity, the body should carry the contents, and headers should communicate metadata. Query parameters may serve as a means to pass header information also in edge cases, but headers are the preferred approach as they are more flexible and can convey more diverse information.

### Require Secure Connections

Require TLS for all API access, without any exceptions. Do not try to determine when TLS is necessary and when it might be skipped — require it universally.

Reject any non-TLS requests by not responding to requests for http or port 80 entirely, preventing any unencrypted data exchange. Where that isn't feasible, respond with a `403 Forbidden`.

Redirects are actively discouraged. They allow sloppy/bad client behaviour without providing any clear gain. Clients that follow redirects increase server traffic twofold, and worse, they render TLS pointless because sensitive information would already have been exposed on the initial non-TLS request before the redirect occurs.

### Require Versioning in the Accepts Header

Managing versioning and transitions between API versions is among the trickier aspects of API design and operation. Put mechanisms in place from the outset.

To avoid unexpected breaking changes for users, it is best to require a version be specified with all requests. Avoid using default versions — they are very difficult, at best, to change in the future.

Specify the version in request headers, alongside other metadata, using the `Accept` header with a custom content type:

```
Accept: application/vnd.heroku+json; version=3
```

### Support ETags for Caching

Include an `ETag` header in all responses, identifying the specific version of the returned resource. This enables caching: users can cache resources and then use requests with this value in the `If-None-Match` header to determine if the cache should be updated.

### Provide Request-Ids for Introspection

Include a `Request-Id` header in each API response, populated with a UUID value. These values should be logged on the client, server, and any supporting services. This creates a mechanism to trace, diagnose and debug requests.

### Divide Large Responses Across Requests with Ranges

For large responses, break them across multiple requests using `Range` headers. This signals when more data is available and how to retrieve it. See Heroku's Platform API documentation on ranges for specifics regarding request and response headers, status codes, limits, ordering, and iteration.

---

## Requests

### Accept Serialized JSON in Request Bodies

Support serialized JSON on `PUT`, `PATCH`, and `POST` request bodies, either as an alternative to or alongside form-encoded data. This establishes consistency with JSON-serialized response bodies.

### Resource Names

Use the plural version of a resource name in most cases. The exception is when the resource in question is a singleton within the system, such as a system's overall status endpoint like `/status`. This keeps it consistent in the way you refer to particular resources.

### Actions

Favor endpoint configurations that avoid needing special actions. However, when actions are necessary, clearly delineate them with the `actions` prefix:

```
/resources/:resource/actions/:action
```

Example — stopping a specific run:
```
/runs/{run_id}/actions/stop
```

Minimize actions on collections. When collection-level actions are unavoidable, use a top-level `actions` path segment to prevent namespace conflicts and make the action's scope explicit:

```
/actions/:action/resources
```

Example — restarting every server:
```
/actions/restart/servers
```

### Use Consistent Path Formats

#### Downcase Paths and Attributes

**Paths:** Use downcased, dash-separated path names for consistency with hostnames. Examples:
- `service-api.com/users`
- `service-api.com/app-setups`

**Attributes:** Attributes should also be downcased, but with underscore separators. This enables attribute names to be typed without quotes in JavaScript. Example:
- `service_class: "first"`

#### Support Non-ID Dereferencing for Convenience

Users may find it cumbersome to supply UUIDs when they think in terms of more human-friendly identifiers — such as a Heroku app name. Accept both an ID or a name:
- `https://service.com/apps/{app_id_or_name}`
- `https://service.com/apps/97addcf0-c182`
- `https://service.com/apps/www-prod`

Do not accept only names to the exclusion of IDs. Names should supplement IDs, not replace them.

#### Minimize Path Nesting

When data models contain nested parent/child resource relationships, URL paths can become deeply nested (e.g., `/orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}`). Limit nesting depth by preferring to locate resources at the root path, using nesting only to indicate scoped collections:

```
/orgs/{org_id}
/orgs/{org_id}/apps
/apps/{app_id}
/apps/{app_id}/dynos
/dynos/{dyno_id}
```

---

## Responses

### Return Appropriate Status Codes

Every response should include the correct HTTP status code.

**Success codes:**
- **200**: Request succeeded for a `GET`, `POST`, `DELETE`, or `PATCH` call that completed synchronously, or a `PUT` call that synchronously updated an existing resource.
- **201**: Used when a `POST` or `PUT` call synchronously created a new resource. Supply a `Location` header pointing to that new resource — especially important for `POST` since the new resource will have a different URL than the original request.
- **202**: Request accepted for a `POST`, `PUT`, `DELETE`, or `PATCH` call that will be processed asynchronously.
- **206**: Request succeeded on `GET`, but only a partial response was returned.

**Authentication & Authorization error codes:**
- **401 Unauthorized**: Request failed because user is not authenticated.
- **403 Forbidden**: Request failed because user does not have authorization to access a specific resource.

**General error codes:**
- **422 Unprocessable Entity**: The request was understood but contained invalid parameters.
- **429 Too Many Requests**: Rate-limiting has been triggered; retry later.
- **500 Internal Server Error**: A server-side failure occurred.

### Provide Full Resources Where Available

Include the full resource representation (i.e. the object with all attributes) whenever possible — on 200 and 201 responses, and for `PUT`, `PATCH`, and `DELETE` requests. On 202 Accepted responses, the full resource representation is not expected.

### Provide Resource (UU)IDs

Give every resource an `id` attribute by default. UUIDs are the recommended default — only deviate if you have a very good reason not to. Avoid IDs that lack global uniqueness across service instances or other resources, especially auto-incrementing IDs.

Render UUIDs in downcased `8-4-4-4-12` format: `"id": "01234567-89ab-cdef-0123-456789abcdef"`

### Provide Standard Timestamps

Provide `created_at` and `updated_at` timestamps for resources by default. These timestamps may not make sense for some resources, in which case they can be omitted.

### Provide Standard Response Types

Define consistent types for all JSON values:

- **String**: string or `null`
- **Boolean**: `true` or `false` only
- **Number**: number or `null`. If you need precision greater than 15 decimals, return a string for that value.
- **Array**: always an array — return an empty array rather than `null` when there are no entries.
- **Object**: object or `null`

### Use UTC Times Formatted in ISO8601

Accept and return times in UTC only, formatted in ISO8601: `"finished_at": "2012-01-01T12:00:00Z"`

### Nest Foreign Key Relations

Serialize foreign key references as nested objects rather than flat key names like `owner_id`. This allows adding more related resource info without restructuring the response. When nesting, serialize either foreign keys only (id, slug) OR the full embedded record. Never provide only a subset of fields — this leads to surprises, confusion, and inconsistencies between different actions and endpoints.

### Generate Structured Errors

Produce consistent, structured response bodies on errors with three fields:
- A machine-readable error `id`
- A human-readable error `message`
- Optionally, a `url` pointing the client to further information about the error and how to resolve it

Example:
```json
{
  "id":      "rate_limit",
  "message": "Account reached its API rate limit.",
  "url":     "https://docs.service.com/rate-limits"
}
```

Document your error format and the possible error `id`s that clients may encounter.

### Show Rate Limit Status

Protect service health by limiting client requests (token bucket algorithm). Return the remaining number of request tokens with each request via the `RateLimit-Remaining` response header.

### Keep JSON Minified in All Responses

Extra whitespace adds needless response size. Many client tools will automatically prettify output for human readability. Optionally provide a way for clients to retrieve verbose responses via a query parameter (`?pretty=true`) or an Accept header parameter (`Accept: application/vnd.heroku+json; version=3; indent=4;`).

---

## Artifacts

### Provide Machine-Readable JSON Schema

Supply a machine-readable schema to exactly specify your API. Use prmd (https://github.com/interagent/prmd) to manage that schema and verify it with `prmd verify`.

### Provide Human-Readable Docs

Provide human-readable documentation that client developers can use to understand your API. Using prmd, you can easily generate Markdown docs for all endpoints with `prmd doc`. Beyond endpoint details, provide an API overview covering:
- Authentication — how to acquire and use authentication tokens
- Stability & versioning — including how to select the desired API version
- Common request and response headers
- Error format
- Multi-language examples of using the API with clients in different languages

### Provide Executable Examples

Supply executable examples that users can copy directly into their terminals — they should work verbatim, to minimize the amount of work a user needs to do to try the API. If you use prmd to generate Markdown documentation, you will get examples for each endpoint for free.

### Describe Stability

Describe your API's stability across its endpoints by indicating their maturity level — for example, through prototype/development/production stage markers. See the Heroku API compatibility policy for a possible stability and change management approach. Once you've declared an API as stable and production-ready, do not make backwards incompatible changes within that API version. If incompatible changes are necessary, introduce a new API with an incremented version number.
