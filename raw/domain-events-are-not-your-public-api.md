---
url: https://codeopinion.com/domain-events-are-not-your-public-api/
date_fetched: 2026-09-15
---

**Sponsor:** Do you build complex software systems? See how NServiceBus makes it easier to design, build, and manage software systems that use message queues to achieve loose coupling. Get started for free.

You finally did it. You stopped publishing CRUD events and started broadcasting domain events.

Instead of publishing `ShipmentStatusChanged`, `ShipmentUpdated`, or `EstimatedDeliveryDateChanged`, you started publishing more meaningful domain events like `DeliveryAttemptFailed`, `ETARecalculated`, and `ShipmentDelayed`.

And apparently that was supposed to be better, right?

Wrong.

## YouTube

Check out my YouTube channel, where I post all kinds of content on Software Architecture & Design, including this video showing everything in this post.

You’re publishing domain events now rather than CRUD events. But you still feel all the pain when you need to change one of those events. You’re breaking consumers, sometimes consumers you didn’t even know existed.

That’s because your domain events are private. They’re really no different than publishing data change events if you’re exposing internal details that the rest of the system shouldn’t know about.

Not every event that occurs within a boundary or service should be something the rest of the system sees.

That’s where people get into trouble.

The general guidance often seems to be that you define domain events and then publish those events to the rest of your system.

That’s not really the case. Your public events are an API.

## Domain Events Describe What Happens Inside a Boundary

Let’s use a shipment in a logistics system as an example.

We might have domain events like:

- `TruckReserved`
- `CarrierAssigned`
- `DispatchInstructionsSent`
- `ETACalculated`
- `ShipperNotified`

All of these events can be useful inside the boundary.

In reality, there would probably be many more of them because what’s actually happening is a workflow. Something kicked off that workflow and multiple things happened as part of it.

But does every other part of the system care about every granular step in that workflow? Probably not. What they care about is the summary of the behavior that occurred.

What was actually happening here? We were dispatching an order.

That’s the event other parts of the system care about. `OrderDispatched`.

They don’t necessarily care that a carrier was assigned, dispatch instructions were sent, the ETA was calculated, and the shipper was notified.

Those are details of how the dispatch boundary accomplished its work.

## Think About Events Like an HTTP API

An easy way to understand this is to compare it to something most developers are already familiar with: an HTTP API.

Let’s say you have a database. You have some database model. It doesn’t matter what type of database you’re using. You have some structure and some shape of data.

Then you may have a domain model that represents that data differently because it contains your business rules and behavior.

Now imagine you’re exposing a public HTTP API that returns JSON.

What do you return? Often it’s a composition of information.

Your database model, your domain model, and the resource model you expose through JSON are not necessarily the same thing.

If you’ve ever made them the same thing, you’ve probably experienced exactly the problem I’m talking about.

If you expose your database schema directly through your API, consumers start depending on that schema. Now when you want to change your database model, you can’t easily do it because you’ve accidentally turned an internal implementation detail into a public contract.

Events are the same thing. They’re an API. They’re a contract.

Your database model is one thing. Your domain model is another thing. Your integration events are your public contract with other parts of the system.

Those do not have to be the same.

## Your Integration Model Is Its Own Model

Imagine we have a shipment.

Internally, we may have a concept like a brokered shipment. We assign a carrier, book the shipment, calculate information about the shipment, and do whatever else the domain requires.

How that shipment is persisted can look very different from the domain model.

But more importantly, neither one necessarily represents what we want to expose outside of that boundary. Other parts of the system might not care that we assigned a carrier. They care that the order was dispatched and brokered.

That’s what our integration event communicates. This is the exact same coupling problem you get when you expose your database internals through CRUD events.

If you’re leaking schema changes through events generated directly from your data model, you’re introducing coupling that’s incredibly difficult to break.

It makes your system brittle. Exposing every internal domain event can cause the exact same problem. You can’t evolve.

## Integration Events Should Communicate Meaning

There can absolutely be overlap between domain events and integration events. The important part is that you’re explicitly deciding what should be communicated outside of the boundary.

You’re trying to communicate when something meaningful happened that another part of the system should know about. Not every granular step of a workflow.

Let’s go back to our shipment.

We publish `OrderDispatched`.

Again, that’s a summary of several things that happened internally. We selected a carrier, booked the shipment, and performed other parts of the workflow.

Other boundaries care that the order was dispatched. Later, the vehicle arrives at the shipper.

Now we might publish `Arrived`.

From there, something unexpected might happen. Maybe the package isn’t ready. Maybe the business is closed. Maybe they don’t need to ship it anymore.

You could say the order was canceled.

But did it really just get canceled?

If other parts of the system now need to infer what happened by looking at a series of generic status changes, you’re missing the actual business concept.

What really happened might be something like `TruckOrderNotUsed`.

The vehicle went to the location where it was supposed to pick up the shipment, but the shipment wasn’t available. That distinction matters.

## The Business Meaning Matters

A truck order not being used has implications throughout the system.

Invoicing might care because the customer still needs to be charged a fee. We actually had a vehicle drive to the location.

Fleet management cares because that vehicle is now available for other work. It can release the vehicle so another shipment can be dispatched to it.

Reporting cares because this situation means something very different from an order simply being canceled.

Settlements care because we may still need to pay the carrier. They didn’t drive there for free.

All of those parts of the system care about what actually happened. Not because an order status changed to canceled. They care because a truck order wasn’t used. That’s a meaningful business concept.

## A Domain Event and Integration Event Can Represent the Same Concept

This is also a good example of where a domain event and an integration event can overlap.

`TruckOrderNotUsed` might be something you care about internally within the dispatch boundary.

Other parts of the system clearly care about it too. So it’s also an integration event.

But that doesn’t mean they need to be the exact same event with the exact same schema.

Let’s say our internal domain event contains:

Maybe `ReasonCode` is some internal value like `789`.

Now things get sketchy. Is that value meaningful outside of your boundary? If another service starts making decisions based on reason code `789`, you’ve leaked an internal implementation detail into your public API.

Now that consumer is coupled to something you considered private.

The domain event and integration event can represent the exact same business concept without having the exact same schema.

It’s about the data you explicitly want to expose.

## How Much Data Should an Integration Event Contain?

This leads to another common question.

How much information should be inside an integration event?

There are generally two ends of the spectrum.

On one side you have really fat events. This is often called event carried state transfer, where the event contains nearly everything about the entity when something changes.

That can quickly turn into exposing your database model through events. On the other side, you have extremely thin events that contain almost nothing except IDs. The problem with events that only contain IDs is that consumers often need more information.

What do they do? They make a synchronous call back to the producer.

Let’s say billing receives `TruckOrderNotUsed`, but the event doesn’t contain all the information billing needs.

Billing now makes an HTTP request back to the shipment or dispatch boundary to get more information. The same issue applies inside a monolith. It could just be an in process call rather than an HTTP request.

There’s a subtle problem with that.

When you make that synchronous call, you’re generally asking for the state right now. You’re not necessarily getting the state from when the event occurred. If the event occurred five minutes ago but the consumer didn’t process it until now, the current state could be completely different.

You’re trying to react to something that happened five minutes ago using information from right now. That’s not always what you want.

So I don’t think this is really about thin events versus fat events. Events shouldn’t contain as little information as possible just because they’re events.

They also shouldn’t contain everything under the sun.

They should contain the information consumers need to understand what happened and react to it.

## Events Tell a Story

This also connects directly to versioning.

Let’s say we started with a very small `TruckOrderNotUsed` event:

Consumers need to understand why it wasn’t used, so maybe we add a `Reason`.

That reason is part of our integration contract. We define the reasons that consumers should understand rather than exposing some internal reason code.

But sometimes what looks like an event versioning problem isn’t really a versioning problem at all.

Sometimes you’ve discovered a different business concept.

For example, `TruckOrderNotUsed` means the truck actually went to the shipper to pick something up and the shipment wasn’t available.

That’s very different from a truck order that was canceled before the vehicle ever left.

Those are two distinct things.

If the truck went to the location, we might still owe the carrier money. If the truck never left, maybe we don’t. If the truck went to the location, maybe the customer gets charged a fee.

If it was canceled before dispatch, maybe they don’t.

Those differences matter to invoicing, settlements, reporting, and other parts of the system.

I could keep changing `TruckOrderNotUsed` trying to represent all of those situations. Or I can recognize that another business concept exists. Maybe that’s `TruckOrderCanceledBeforeShipment`.

Now our events are telling a much clearer story.

## Treat Events Like a Public API

Treat your integration events like a public API. There really isn’t a difference. They’re a contract.

Defining that contract explicitly gives you the ability to evolve your system.

You don’t have to decide that some domain event you created today can never change because consumers might depend on it forever.

Your domain events are internal.

Can you decide that one of those events is incredibly stable and expose the same concept externally? Sure.

It might even have the exact same name. But it doesn’t necessarily need to contain the exact same data.

What you need internally and what you want to expose externally are two different decisions.

Your data model is not your integration model. Your domain model is not your integration model either.

You get to explicitly decide what your public API is. You get to decide what your public events are. And that distinction gives you room to change everything behind that contract without dragging every consumer along with you.

	**Join CodeOpinon!**

	Developer-level members of my Patreon or YouTube channel get access to a private Discord server to chat with other developers about Software Architecture and Design and access to source code for any working demo application I post on my blog or YouTube. Check out my Patreon or YouTube Membership for more info.
