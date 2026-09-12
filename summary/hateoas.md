---
url: https://htmx.org/essays/hateoas/
title: HATEOAS
author: htmx.org
date_fetched: 2026-08-08
topics:
  - software-engineering-craft
---

# HATEOAS — Summary

A reworking of the Wikipedia entry on HATEOAS (Hypermedia as the Engine of Application State), using HTML rather than JSON to explain the REST constraint. The essay argues that HATEOAS is the defining characteristic that distinguishes REST from other network architectures: a client interacts with a server entirely through hypermedia responses, discovering all available actions from the responses themselves rather than from out-of-band documentation.

The core example walks through a bank account resource. In the HTML/hypermedia version, the server returns different sets of available links depending on account state (positive balance → deposit, withdraw, transfer, close; overdrawn → deposit only). The client needs no prior knowledge of what "overdrawn" means — it simply renders the links the server provides. A JSON API, by contrast, returns a `status: "overdrawn"` field and requires the client to know what that means, what actions are available, and what URLs to use — all from out-of-band documentation.

The essay traces HATEOAS to Roy Fielding's doctoral dissertation on the early web architecture (HTML + HTTP) and critiques the industry's appropriation of "REST" to describe JSON APIs that lack hypermedia controls. The Richardson Maturity Model placed "Hypermedia Controls" at Level 3, but attempts to bolt hypermedia onto JSON (via `links` properties) failed because JSON isn't a natural hypermedia — it can't encode HTTP methods, expected inputs, or form semantics the way HTML's `<form>` element can. The essay concludes that a natural hypermedia like HTML is a practical necessity for building genuinely RESTful systems.
