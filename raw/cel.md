---
url: https://cel.dev/
title: Common Expression Language (CEL)
author: Google
date_fetched: 2026-07-18
date_published: unknown
---

# Common Expression Language (CEL)

CEL is an expression language that's fast, portable, and safe to execute in performance-critical applications. It is designed to be embedded inside larger applications, supporting application-specific extensions. Google is the creator and steward — the spec lives at `github.com/google/cel-spec`.

## What CEL Is

CEL is described as "an expression language that's fast, portable, and safe to execute in performance-critical applications." It is designed to be embedded inside larger applications, supporting application-specific extensions.

## Syntax Overview

The page shows four code examples:

1. **Simple predicates** — e.g., `'tacocat'.startsWith('taco')`
2. **Parameterized predicates over structured data** — e.g., `account.balance >= transaction.withdrawal`
3. **JSON objects** — e.g., `{'sub': '12345678', 'aud': 'example2.cel.dev', 'iss': 'https://...'}`
4. **Strongly typed objects** — e.g., `common.GeoPoint{ latitude: 10.0, longitude: -5.5 }`

## Core Features (4 pillars)

| Feature | Description |
|---|---|
| **Fast** | "Accelerated expression evaluation in performance-critical paths from nanoseconds to microseconds." |
| **Portable** | "Developer friendly, light weight with common syntax across multiple Google and external systems." |
| **Extensible** | Supports subsetting and extension; easy to embed and tailor to configuration and policy needs. |
| **Safe** | Non-Turing complete; "only accesses data provided by the host application." |

## Use Cases

CEL is recommended for:
- List filters for API calls
- Validation constraints on protocol buffers
- Authorization rules for API requests

## Is CEL Right for Your Project?

The page explains that CEL is "especially useful for predicate logic and simple data transformations." It is most efficient when expressions are "evaluated frequently, but modified infrequently." A key example: evaluating an HTTP request against a security policy — "a one-time configuration cost for validating the expression" followed by frequent evaluation at negligible cost.

## Performance Profile

Evaluations complete in the **nanoseconds to microseconds** range with predictable costs, making it suitable for performance-critical paths.

## Contribution

The project is open source. The page invites contributions to "our open source code and documentation" via GitHub.

## Where to Learn More

- **Overview**: cel.dev/overview/cel-overview
- **Get Started Tutorial**: cel.dev/tutorials/cel-get-started-tutorial
- **Language Definition**: github.com/google/cel-spec/blob/master/doc/langdef.md
- **GitHub**: github.com/google/cel-spec
- **Contact**: cel-lang-configuration@googlegroups.com (Google Groups email)
