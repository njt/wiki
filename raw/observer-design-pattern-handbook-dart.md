---
url: https://www.freecodecamp.org/news/the-observer-design-pattern-handbook-event-driven-architecture-domain-driven-design-in-dart/
title: "The Observer Design Pattern Handbook: Event-Driven Architecture & Domain-Driven Design in Dart"
author: Oluwaseyi Fatunmole
date_fetched: 2026-07-18
date_published: 2026-07-16
site: freeCodeCamp.org
---

# The Observer Design Pattern Handbook: Event-Driven Architecture & Domain-Driven Design in Dart

By Oluwaseyi Fatunmole, published July 16, 2026 on freeCodeCamp.org.

## Introduction

The article opens by describing a fundamental software challenge: when something happens, multiple parts of a system need to react. Examples include a user login triggering token saving, profile caching, analytics, and navigation; a payment confirmation updating inventory, issuing receipts, and triggering fulfillment; or a sensor change updating multiple UI panels simultaneously.

The "naïve solution" is centralizing all logic in one place. This works initially but becomes brittle as requirements evolve. The Observer pattern provides "a structured, production-grade way to say: when this event happens, notify everyone who cares, without the event source knowing who those people are."

The handbook promises to teach the pattern from first principles, with Dart implementation, connections to Event-Driven Architecture, and integration with Domain-Driven Design and Riverpod in Flutter.

## Table of Contents

Pattern definition, the problem it solves, core components, Dart implementation, a login flow example, a generic EventBus, existing Observer usage in Flutter, Event-Driven Architecture deep dive, DDD application, Riverpod integration, testing, when to use/not use the pattern, and a conclusion.

## What Is the Observer Design Pattern?

The Observer pattern is "a behavioural design pattern that defines a one-to-many dependency between objects." When one object changes state, all dependents are notified automatically.

The author uses a newspaper subscription analogy: "The publisher is called the Subject. The subscribers are called Observers. The newspaper is the event."

The pattern was "formally defined in the Gang of Four book, Design Patterns: Elements of Reusable Object-Oriented Software."

## The Problem It Solves

The article presents a login function performing four tasks inside one method — saving tokens, caching user data, navigating, and tracking analytics. The issue is "tight coupling. The login logic is coupled to every single consequence of a successful login."

Every new requirement forces modification of that single function, risking breakage of all other responsibilities. The Observer pattern "breaks these couplings completely. The login logic does one thing: it performs the login and announces the result."

## Core Components

Four building blocks are described:

- **Subject:** Holds observers and notifies them when events occur. "It just delivers it."
- **Observer:** An interface defining the contract observers follow.
- **Concrete Subject:** Manages the observer list, subscriptions, and notifications.
- **Concrete Observers:** Classes implementing the Observer interface, each with "a specific, focused job."

## Implementing Observer in Dart

**Step 1 - Observer Interface:** An abstract class `LoginObserver` with `onLoginSuccess(UserDto user)` and `onLoginFailed(AppException error)` methods.

**Step 2 - Subject Interface:** An abstract class `LoginSubject` with `subscribe`, `unsubscribe`, `notifySuccess`, and `notifyFailure` methods.

**Step 3 - Concrete Subject:** A `LoginService` class implementing `LoginSubject`, maintaining a `List<LoginObserver>` and using two key design decisions:

1. **Snapshot iteration** with `List.of(_observers)` to prevent `ConcurrentModificationError` when observers unsubscribe during notification loops.

2. **Per-observer try/catch** so that if one observer throws, others still execute. The author warns that "without this, one failing observer would stop the entire notification chain."

## A Real-World Example: The Login Flow

**LoginLogic** class takes a `LoginSubject` and `AuthRepository` as constructor dependencies, depending on the abstraction not the concrete implementation. Its `callLogin` method calls the repository and notifies success or failure accordingly. "Its entire responsibility is: perform the login, announce the result."

**Concrete Observers:**
- `TokenObserver` — saves token on success, deletes stale token on failure
- `UserObserver` — saves user cache on success, clears on failure
- `NavigationObserver` — navigates to home on success, shows error on failure. Uses an injected `NavigationService` abstraction rather than `BuildContext`
- `AnalyticsObserver` — fires appropriate analytics events

Each "has exactly one responsibility. Each one has exactly one reason to change."

**Wiring:** A `setupLogin` function creates the `LoginService` and uses the cascade operator to subscribe all observers. Adding a fifth observer means "creating the class and adding one line here."

## Making It Production-Grade with a Generic EventBus

The article introduces `DomainObserver<T>`, a generic observer where "the type parameter T represents the data type the observer expects on success." An `EventBus<T>` class manages typed observers with the same snapshot iteration and error isolation.

"Now every feature gets the same infrastructure without duplicating a single line of the pattern." Examples: `loginBus`, `paymentBus`, `orderBus` — each typed to its domain concept.

## Observer Is Already in Your Flutter Code

The article points out that developers have been using the Observer pattern without realizing it:

- **Streams and StreamController:** "StreamController is a Subject. stream.listen is subscribe. sink.add is notifyObservers."
- **ChangeNotifier:** "notifyListeners() iterates over every registered listener and calls them. Those listeners are Observers."
- **BLoC:** "The BLoC is the Subject. The builders and listeners are Observers."

The author states that "Flutter's entire reactive system (Streams, ChangeNotifier, BLoC, ValueNotifier) is the Observer pattern with lifecycle management built in."

## Deep Dive Into Event-Driven Architecture

**What is EDA?** A design paradigm where "the flow of the application is determined by events." Components communicate by producing and consuming events through a shared bus rather than calling each other directly.

The article contrasts request-driven flow (A calls B directly, waits) with event-driven flow (A publishes to EventBus, which delivers to B, C, D). "Component A doesn't know about B, C, or D."

**Events Are Facts, Not Commands:** A command says "do this" and expects a response. An event says "this happened" — it's "an immutable record of a fact." When modeling with events-as-facts, the system "becomes auditable and predictable in ways that command-driven systems are not."

**Domain Events in Dart:** Events should be immutable value objects. A `DomainEvent` base class includes `occurredAt` timestamp and `eventId` identifier. Concrete events like `UserLoggedIn` and `LoginFailed` extend it.

**Type-Safe DomainEventBus:** An `EventHandler<T extends DomainEvent>` interface with a `handle(T event)` method. The `DomainEventBus` uses a `Map<Type, List<EventHandler>>` where "the key is a Type (the event class itself)." The `register<T>` and `publish<T>` methods are generic. When published, "only its registered handlers fire."

## Application in Domain-Driven Design

**Key DDD Concepts:**
- Domain Events are "first-class citizens in DDD" representing meaningful business occurrences
- Aggregates are "the natural source of domain events" — they enforce business rules and raise events
- Use Cases orchestrate: they call repositories, get results, raise domain events, return outcomes — they don't handle side effects directly

**Clean Architecture Structure:** The domain layer is pure Dart with "zero framework dependencies." The folder structure separates core events, feature-specific domain events/handlers/entities/repositories/usecases, data layer implementations, and presentation providers/pages.

"The critical rule: the domain layer is pure Dart. No Flutter imports. No Riverpod imports. No HTTP imports."

**Login Use Case:** A `LoginUseCase` receives `AuthRepository` and `DomainEventBus` as dependencies. On success it publishes `UserLoggedIn` and returns `Result.success`. On failure it publishes `LoginFailed` and returns `Result.failure`. "The use case doesn't know how many handlers are registered."

## The Riverpod Hybrid: Clean Architecture in Practice

**The Problem Being Solved:** Two common pain points with Riverpod:

1. "Fat ref.listen in widgets" — side effects in widgets tied to widget lifecycle
2. "Fat notifiers" — a notifier with six responsibilities violates SRP

**The Clean Rule:** "The use case owns domain consequences. The notifier owns UI state. Widgets own nothing." Handlers fire independently of widget lifecycle. The architecture "works correctly whether login is triggered from a widget, a biometric prompt, a deep link, or a background service."

**Understanding AsyncNotifier:** Riverpod 2.0's `AsyncNotifier` holds an `AsyncValue<T>` which can be `AsyncData`, `AsyncLoading`, or `AsyncError`. With code generation via `@riverpod` annotation, the provider and boilerplate are auto-generated.

**The Thin Notifier:** A `LoginNotifier` with `@riverpod` annotation extends `_$LoginNotifier`. The `build` method returns `AsyncData(null)` (initial state). The `login` method sets `AsyncLoading`, calls the use case, then sets `AsyncData` or `AsyncError` based on the result. "It does exactly one thing: reflect the outcome of the use case as UI state." No token saving, navigation, caching, or analytics appear here — those are already handled by the event bus.

**The Widget:** A `LoginPage` uses `ref.watch(loginNotifierProvider)` and `loginState.when` to render form, loading spinner, or error view accordingly. "The widget knows nothing about tokens, navigation, or caching."

**Wiring the Composition Root:** A Riverpod provider creates the `DomainEventBus`, registers all handlers with injected dependencies, and returns the bus. The `LoginUseCase` provider receives both `authRepositoryProvider` and `eventBusProvider`. "Each layer knows only about the layer directly below it and nothing else."

Adding a new side effect means "creating a new handler class and adding one bus.register line in the composition root."

## Testing the Observer Architecture

**Use Case Tests:** Mock the repository and event bus. Verify the correct event type is published for success and failure outcomes. "It doesn't test what any handler does."

**Handler Tests:** Each handler test is tiny — create the handler with mocked dependency, fire the event, verify the exact side effect. "No other handler is involved, no notifier is involved, and no widget is involved."

**Notifier Tests:** Only verify state transitions (loading→data on success, loading→error on failure). No event bus mocking needed since "the notifier no longer touches the event bus."

## When to Use the Observer Pattern

Use it when one event triggers multiple independent reactions, when reactions need adding/removing without modifying the event source, when side effects need decoupling from business logic, when each reaction should be independently testable, when multiple parts react to the same state change, or when a feature will grow in number of side effects over time.

## When Not to Use It

Avoid it when there's only one consumer with no realistic expectation of more, when the producer-consumer relationship is simple and direct, when "the pattern adds structural overhead without meaningful benefit," when existing Flutter reactivity (Streams, ChangeNotifier, Riverpod) already solves the problem, or when strict ordering of side effects is critical and fan-out makes that hard to guarantee.

## Conclusion

The Observer pattern "solves a problem every growing application faces: how do you let one event trigger many reactions without turning your codebase into a tightly coupled mess."

The article recaps building the pattern step by step in Dart with snapshot iteration, per-observer try/catch, and dependency inversion. It notes the pattern is already embedded in Flutter's reactive systems. It then traces the journey through Event-Driven Architecture (with events as immutable domain facts), Domain-Driven Design (with a pure Dart, framework-independent domain layer), and Riverpod integration under a clear rule: "handlers own side effects, the notifier owns UI state, and widgets own nothing."

The result is a codebase that scales gracefully: "When a new side effect needs to be added, you create one handler and register it in one place. Nothing else changes."

The article closes with "Happy Coding!"
