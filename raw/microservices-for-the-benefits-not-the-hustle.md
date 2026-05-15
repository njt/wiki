---
url: https://wolfoliver.medium.com/the-purposes-of-microservices-4e5f373f4ea3
title: "Microservices for the Benefits, Not the Hustle"
author: Oliver Wolf
date_fetched: 2026-05-15
date_published: 2023-01-31
---

# Microservices for the Benefits, Not the Hustle

Author: Oliver Wolf
Published: January 31, 2023 (Medium, 11 min read)

## Core Premise

Wolf addresses a common misconception: people often claim a monolith "cannot be scaled" and must be rebuilt as microservices. He argues scalability is the first benefit people think of, but for most systems, the break-even point is far off. Despite this, he remains "a believer in microservices, even if you do not have a big load on your system."

Requirements must be defined and ranked *before* building architecture, as the optimal microservice design varies by priority.

## Purpose 1: Minimize Costs of Change

"the major benefit of the microservice architecture style," rooted in the single responsibility principle. Rules:

- Different responsibilities go in different services
- Each service gets its own code repository
- Each microservice instance runs as a dedicated process
- Inter-service communication only through network + official APIs
- "each service must have its own logical database schema"
- Protocols must be technology-agnostic
- Choreography preferred over orchestration

### Enforced Cohesion via Hard Boundaries

Because responsibilities live in separate repos, it becomes much harder for developers to place code where it doesn't belong -- a common problem in traditional apps where "developers will eventually ignore or oversee the boundaries of subsystems due to time pressure."

### Lower Coupling via Choreography

Orchestration creates point-to-point connections forming a tangled web; choreography means each service "observes its environment and acts on events autonomously." Services subscribe to a message bus. New services are added simply by connecting to the bus. For CRUD operations, REST is still appropriate.

### Smaller Code Bases

Enables faster onboarding and easier dead-code removal. Cites tweet about "more source code causes many more errors" and lighter note that "the IDE loads faster."

### Smaller Teams

Teams organized around individual services. References source claiming small teams "minimize management overhead" and "increase productivity dramatically."

### Interim Conclusion

Quotes *Building Microservices* by Sam Newman: "Applying a microservice architecture is not about building the perfect system instead it is about building a framework in which a good system can emerge over time as the understanding grows."

Warns against "nanoservices" antipattern: when services are "too big or too small, the advantages are gone and problems arise."

## Purpose 2: Encourage Generalization, Replaceability, and Reuse

Reuse is a fundamental SOA goal carried into microservices. Principle: "Build smaller services that do one thing well."

### Sizing Rules of Thumb

- Jon Eaves: "The service can be rewritten and redeployed in 2 weeks."
- Werner Vogels (Amazon): "It must be possible to feed a team that maintains a service with two pizzas."

Wolf reframes: instead of asking *how big*, ask **which responsibilities should go into the same service?** Answer: group responsibilities so that "amount and size of domain-specific services are minimized," maximizing reusable services.

### Decomposition Dimension 1: Actions on Entities

CRUD operations on the same database records should live in the same service's API. Strongly related entities are good candidates for a single service.

### Decomposition Dimension 2: Aspects of an Action

Examples:
- Email sending after user signup: delegate to a generic email service
- User-specific advertising: use off-the-shelf ad systems rather than bloating the product service. Amazon reportedly calls "about 200 services" when a user opens a page
- Fitness tracker data: a "heart rate service" should generalize to a time-series database

## Purpose 3: Increase Operations Efficiency

Microservices have a larger total footprint since each runs as a separate process. However, for horizontal scaling, monolithic apps require duplicating the entire application. With microservices, "you can only scale those parts that actually have performance issues."

## Risk 1: Increase Operations Complexity

"Many more applications than just one or two" must be operated. Mitigations: containers (Docker), PaaS (Cloud Foundry), continuous delivery.

## Risk 2: Distributed Monolith

If cohesion is poor, distributing across microservices "will multiply all problems."

## Key Quotes

- "Making a system scalable -- or cheaper to scale -- is the first benefit that comes to mind when you think about microservices."
- "different responsibilities are placed in different services"
- "each service must have its own logical database schema"
- "choreography should be preferred over orchestration"
- "Each microservice architecture is a service-oriented architecture, but not the other way around."
- "Hard system boundaries" enforce cohesion and prevent boundary violations.
- "The service can be rewritten and redeployed in 2 weeks."

## Themes

1. Scalability is overrated as a reason -- most systems don't have the load to benefit
2. Changeability is the real prize -- microservices excel at agile, incremental evolution
3. Service sizing is critical -- both too big and too small create problems
4. Reuse drives efficiency -- generic problems solved by reusable/generic services
5. Choreography beats orchestration -- event-driven communication reduces coupling
6. Microservices != SOA, but a refinement of it -- smaller size and lightweight protocols are the differentiators

## External Sources Referenced

*Software Architecture in Practice* (Bass, Clements, Kazman), *Building Microservices* (Sam Newman), *System Analysis and Design in a Changing World* (Satzinger), Werner Vogels, Jon Eaves
