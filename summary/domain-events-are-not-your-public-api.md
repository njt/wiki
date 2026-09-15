---
url: https://codeopinion.com/domain-events-are-not-your-public-api/
title: "Domain Events Are Not Your Public API"
author: Derek Comartin
date_fetched: 2026-09-15
topics:
  - software-engineering-craft
  - distributed-systems
---

Derek Comartin (CodeOpinion) argues that graduating from CRUD events to domain events doesn't fix coupling if you publish every internal domain event to the rest of the system. Domain events describe what happens *inside* a boundary — `TruckReserved`, `CarrierAssigned`, `ETACalculated` — and most of them are workflow details other parts of the system shouldn't see. What consumers actually care about is the summary of behaviour: `OrderDispatched`.

The core move is treating integration events as a public API, exactly analogous to an HTTP resource model: your database model, your domain model, and your integration model are three different things, and collapsing them turns implementation details into contracts you can never change. A domain event and an integration event can represent the same concept (`TruckOrderNotUsed`) without sharing a schema — leaking an internal `ReasonCode: 789` couples consumers to private decisions.

On event payload design, he rejects both extremes: fat "event carried state transfer" events re-expose the database, while ID-only thin events force synchronous call-backs that fetch *current* state rather than the state at event time. Events should contain what consumers need to understand what happened. Finally, some apparent versioning problems are really undiscovered business concepts — a truck that arrived but couldn't load is a different event than one cancelled before dispatch, because invoicing, settlements, and fleet management treat them differently.
