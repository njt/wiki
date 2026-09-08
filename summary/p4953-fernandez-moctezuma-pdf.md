---
url: https://www.vldb.org/pvldb/vol19/p4953-fernandez-moctezuma.pdf
title: "The Dataflow Model Revisited"
author: Tyler Akidau, Rafael J. Fernández-Moctezuma, Reuven Lax, Daniel Mills
date: 2026
date_fetched: 2026-09-08
site: vldb.org
---

# The Dataflow Model Revisited

On the occasion of its VLDB Test of Time award, the four authors of the 2015 Dataflow Model paper grade their own work: what aged well, what aged badly, and what they missed. Their verdict in brief — *we got the physics right, but the interface wrong.*

## What aged well

Three bets held up essentially without caveat, and all three apply across streaming generally, not just analytics: **event time versus processing time** (the distinction is now load-bearing vocabulary in every major system); **consistency is non-negotiable** (the Lambda Architecture's weakly-consistent speed layer is dead, exactly-once is table stakes); and **never rely on completeness** (refuse to groom unbounded data into a finite pool that eventually becomes complete).

## What aged badly

The interface. **Windowing** proved to be just another grouping dimension ("a time-flavored GROUP BY"), not a consistency mechanism — promoting it to the center of the model was a category error, and the formalism even missed the *splitting* inverse of merging (temporal validity windows that shrink). **Triggers** were an over-engineered answer to a question users should never have faced; a decade of production distilled the trigger menagerie to two members (fire on watermark, fire periodically), and the construct that survived is *target lag* — a single declared freshness bound the engine optimizes against. **User-facing retractions** were barely used as designed; the mechanism was right but the altitude was wrong (it's the engine's business in table form, the contract in stream form). **Ordering** was never promised but always required; **latency** was oversold (the dial can't reach milliseconds at scale — demand bifurcates along the old OLTP/OLAP line).

## What they missed

The biggest miss: **tables**, and the streams-and-tables duality. A stream is a changelog, a table is a point-in-time snapshot, and each is recoverable from the other. The 2015 paper drew both halves repeatedly and missed the nouns. Had they seen it, retractions become ordinary changelog rows, triggers become a materialization policy, and the "unified model" arrives by recognizing batch and streaming are views of the same thing. The deeper truth they now state plainly: *the problem the 2015 paper attacked with windows, triggers, watermarks, and retractions is the materialized-view maintenance problem* — posed and theorized by the database community thirty years earlier, left unfinished by them, underappreciated by the authors, and completed at last by both communities together. The destination for analytical streaming is to disappear into the database: the user declares a query and a freshness contract, the engine streams, and the user never sees it.
