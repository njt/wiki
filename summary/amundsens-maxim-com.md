---
url: https://www.amundsens-maxim.com/
title: "Amundsen's Maxim"
author: Mike Amundsen
date_fetched: 2026-10-26
date_published: unknown (maxim first proposed 2012)
topics:
  - software-engineering-craft
---

This site is a single-purpose shrine to **Amundsen's Maxim**: "Remember, when designing your Web API, your data model is not your object model is not your resource model is not your message model." The maxim, first proposed by Mike Amundsen in 2012, says that the four models of an API — how data is stored, how it is structured in code, how it is exposed to clients, and how it travels over the wire — are distinct concerns and must be designed independently.

The page breaks the maxim into three separations. **Data ≠ object**: a 1:1 mapping between database schema and code objects couples the two tightly. **Object ≠ resource**: resources should be shaped around client use cases, not internal object structures; mapping objects directly to resources yields rigid, CRUD-shaped APIs. **Resource ≠ message**: payloads should be designed for network efficiency and real client needs, not as raw dumps of resource representations.

The payoff claimed is flexibility and evolvability: if each layer is decoupled, changes in one don't force changes in the others, and APIs stop breaking whenever the database schema moves. Amundsen traces intellectual lineage to Fielding's REST constraints and Eric Evans's Bounded Contexts from DDD. The site also lists citations showing the maxim circulating in practice (CodeOpinion, Nordic APIs, APIscene, StackOverflow threads) and a short biography of Amundsen, whose books include *RESTful Web APIs* (with Leonard Richardson) and *Continuous API Management*.
