---
url: https://wolverinefx.io/tutorials/vertical-slice-architecture.html
title: "Vertical Slice Architecture"
author: JasperFx Software (Wolverine)
date_fetched: 2026-08-26
site: wolverinefx.io
topics:
  - software-engineering-craft
---

# Vertical Slice Architecture

By JasperFx Software (the Wolverine maintainers), publication date not stated on the page. The article is written from a CQRS standpoint.

## Summary

A tutorial on applying Vertical Slice Architecture (VSA) — organizing code by feature or use case rather than by horizontal technical layer — using the Wolverine .NET framework. The authors position Wolverine as a more idiomatic alternative to the MediatR-based VSA that dominates the .NET ecosystem, arguing you get better results by leaning into Wolverine's capabilities (side effects, cascading messages, HTTP endpoints, transactional middleware) than by using it as a drop-in MediatR replacement.

The guiding philosophy has three pillars: effective test coverage is paramount, code should be easy to reason about, and iteration should be easy. The recommended approach is the "A-Frame Architecture," which splits code into three responsibilities — business logic, infrastructure services, and coordination/controller logic sitting on top — rather than Clean/Onion Architecture's layered abstractions.

## Key Points

- **A-Frame over Clean/Onion**: isolate behavioral logic from infrastructure by dividing code into business logic, infrastructure services, and coordinating logic — not by technical layers with repository abstractions everywhere
- **Pure-function handlers via side effects**: Wolverine's `Insert<T>`/`Delete<T>` "Storage Side Effect" types and cascading messages let handlers return declared effects instead of calling infrastructure directly, keeping them unit-testable without mocks
- **No repository abstractions**: the authors recommend using Marten/EF Core APIs directly in handlers and shrinking call-stack depth, arguing simple code beats future-proofing against persistence-tooling swaps
- **Wolverine.HTTP endpoints**: built-in route discovery and OpenAPI metadata extraction replace the "Minimal API delegating to MediatR" pattern, with `Validate()` compound handlers as a low-ceremony Railway Programming for the happy path
- **Transactional outbox for free**: durable local queues + inbox storage mean the outgoing `OrderCancelled` event survives crashes, giving resilience with low code ceremony
- **Query side is thin**: they advise against MediatR-style query handlers; just use raw persistence tooling directly in GET endpoints and rely on integration tests

## Code Examples

Walked through a `PlaceOrder` command (transaction-script style, then refactored to a pure function returning `Storage.Insert(order)`) and a `CancelOrder` HTTP endpoint using `[WolverinePost]`, `[Entity]`, `[EmptyResponse]`, and a `Validate()` method returning `ProblemDetails`. Recommended layout: one file per command/query/endpoint containing the message type, an inner FluentValidation validator, and the handler/endpoint class.

## Target Audience

.NET developers using (or considering) CQRS, MediatR, or Clean Architecture who want a concrete alternative that reduces ceremony and improves testability.
