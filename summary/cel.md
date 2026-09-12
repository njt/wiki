---
url: https://cel.dev/
title: "Common Expression Language (CEL)"
author: Google
date_fetched: 2026-07-18
topics:
  - developer-tools
---

CEL is an embeddable expression language from Google, designed for fast,
portable, and safe expression evaluation in performance-critical applications.
It is non-Turing complete and can only access data explicitly provided by the
host application.

The language targets predicate logic and simple data transformations, with
evaluation times in the nanosecond-to-microsecond range. Expressions compile
once and evaluate repeatedly at negligible cost — ideal for patterns like
security policy checks on HTTP requests, API filter expressions, and protocol
buffer validation.

CEL is open source, with its specification and implementations hosted at
`github.com/google/cel-spec`. It is already used across multiple Google and
external systems.
