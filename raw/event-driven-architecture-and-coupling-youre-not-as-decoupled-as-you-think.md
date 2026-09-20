---
url: https://codeopinion.com/event-driven-architecture-and-coupling-youre-not-as-decoupled-as-you-think/
date_fetched: 2026-09-20
---

**Sponsor:** Do you build complex software systems? See how NServiceBus makes it easier to design, build, and manage software systems that use message queues to achieve loose coupling. Get started for free.

So, you broke apart your monolith. You introduced a message broker. You’re using event driven architecture. Events are flying everywhere.

You’re decoupled now, right?

But somehow nothing feels any better because your system is still really brittle.

## YouTube

Check out my YouTube channel, where I post all kinds of content on Software Architecture & Design, including this video showing everything in this post.

## CRUD Events

The problem might be the events you’re publishing:

`ShipmentStatusChanged``ShipmentAddressUpdated``EstimatedDeliveryDateChanged``CarrierIdUpdated`

Every other service subscribes to these events and maintains its own local copy of shipment data.

You might think you’re decoupling your system because you’re publishing events to different consumers, but you’re not. All you’ve really done is start replicating data everywhere.

## What Does Shipment Status Changed Actually Mean?

Suppose we introduce a new shipment status.

We already have an event called `ShipmentStatusChanged`, but now we’re left wondering who consumes it. Do those consumers really understand what all the statuses mean? If I add a new one, am I going to break them?

What does `ShipmentStatusChanged` even mean?

The status changed. Great.

What are the possible statuses? Is there a progression? If a new status exists and I don’t know about it, what happens?

I don’t want consumers to implicitly understand or reverse engineer what’s going on inside my data model.

There’s a big difference between how your system is coupled when you’re broadcasting data changes or state changes versus publishing explicit events about what actually happened in your system.

What I’m really talking about here are CRUD events.

It’s CRUD over pub/sub.

Instead of saying:

`ShipmentStatusChanged`

What I really care about is being explicit about the actions that occurred:

`ShipmentDispatched``ShipmentDelayed``ShipmentDeliveryAttempted``ShipmentLoaded``ShipmentDelivered`

It’s not about a status changing. It’s about what actually happened.

Those events have meaning. Something happened that is important within the system.

Does that mean a status change isn’t important? No. But it isn’t explicit.

And it’s really easy to fall into this trap because it can have some severe consequences.

## Your Internal Data Model Becomes a Public API

If we have something like a shipment, it’s easy to think that whenever one of its properties changes, we should publish an event.

The shipment weight changed. The shipment destination changed. The shipment status changed.

The problem is that you’ve taken your internal data model and turned it into a public API.

It’s really easy to think that because you’re using event driven architecture and asynchronous messaging, you’re decoupled.

You’re not.

There’s really not much difference between publishing all these data model change events to your services and allowing those services to understand the internals of another service’s database.

Yes, you removed the synchronous call from service to service. Maybe you no longer have an HTTP API call directly between them.

You removed that temporal aspect of the coupling. But you did not remove the coupling. You’re coupled through the messages and, more specifically, through the semantics of those messages and how you’ve designed them.

Suppose we publish `ShipmentStatusChanged` and the new status is `4`.

What does status `4` mean?

A consumer might understand that `4` means delivered. So it has some code that processes the message and says that if the status changes to `4`, it needs to send a delivery notification.

Maybe that’s an email. Maybe it’s something else.

But what happens if we change what those statuses mean? We can’t.

Our consumer understands that `4` means delivered. 

What happens if the workflow changes and `4` no longer means delivered? Again, we can’t just change it because we’re going to break our consumers.

The consumer understands our internals. It understands how we store state and that the value `4` means delivered. Now we’re stuck with that. And I get it. People are probably thinking that the problem is using an integer. We should use a string, an enum, or some constant instead.

That’s not the point.

The point is that the consumer has to infer what something means or explicitly understand the internals of another service. Changing the data type doesn’t fix that.

## Consumers Should Not Reverse Engineer Your State

It’s not wild to think that a consumer could subscribe to several different events and start inferring business behavior from them.

I’ve seen this before.

Maybe the consumer sees that the shipment location changed. Then the scheduled delivery date changed. Then some other property changed.

Based on those three events, it infers that the shipment must have been delayed.

That seems logical, right? But was it actually delayed? We never explicitly said that.

The consumer inferred it by understanding a combination of state changes inside another service. That’s the problem. Your consumers shouldn’t have to infer or reverse engineer your state changes and data model to understand what’s actually happening.

If the shipment was delayed, publish an event that says the shipment was delayed.

Be explicit.

## Domain Events and Integration Events Are Different

There’s another distinction that matters here, which is the difference between domain events and integration events.

What I’m really talking about in this post are integration events. One way I like to describe the difference is inside events versus outside events. Domain events are inside events. They exist inside my particular service or service boundary. Anything going outside that boundary is an outside event.

Another way to think about it is internal versus external. An internal event is something I control. I can mutate it. I can change it because I’m changing everything within that service boundary. An external event is different.

Now I have a contract.

I have consumers using it. I probably need to version it. I can’t just change it whenever I want because I’m going to break people. Or think about it the same way you think about APIs.

Domain events are private. They’re for me. They live within my local boundary. Integration events are public. They’re contracts.

If you’ve ever built an HTTP API, you already understand this.

If something is public facing, you can’t just change it. You have to think about versioning and backwards compatibility.

Events are no different.

## Model Events Around Business Concepts

Model your events around business concepts.

I like to think about the things the business actually cares about. These are the things you hear people talking about where everybody understands what happened.

The shipment was dispatched. The shipment was delayed. Delivery was attempted. The shipment was delivered.

Those are things actually occurring in the system. That’s what you want to model events around. There is still a distinction based on how you’re using those events.

Inside events are often more granular. They’re more fine grained and may contain data that’s completely reasonable to expose within that boundary because the context is understood.

They’re still behavior centric. They represent something happening in the system.

But they’re internal. You own them. You can change and evolve them because they live entirely inside that service.

Outside events are different. When you expose events to other services, they usually shouldn’t be as granular. You probably don’t want to expose every minor thing that happens internally.

Often an integration event is more of a summary. Maybe a combination of things occurred and you’ve reached some meaningful checkpoint.

Now you can say, yes, this happened. It’s still behavior centric, but it represents something meaningful that other services actually care about. Treat those events like an API. Version them. Think about backwards compatibility. If consumers depend on an event and you change it in an incompatible way, you’re going to break them.

## Give Consumers the Event They Actually Want

Once you start being explicit about your events and modeling them around things the business actually cares about, you usually end up with exactly what consumers wanted in the first place.

Consumers want predictability.

If a consumer needs to send an email when a shipment is delivered, it wants to know that the shipment was delivered.

It doesn’t want to receive an event saying the order status changed to `4`, then understand that `4` means delivered, and then decide what to do.

Let it consume a specific event for a specific use case.

`ShipmentDelivered`.

That’s a little easier to understand than inspecting a status and having to know the internal model of another service.

Just be explicit.

## What About Distributing Data?

I’ve been ragging on distributing data around via events, but there is a use case for it.

Reporting is a good example.

There’s a giant caveat here, though, especially when we start talking about CDC, or change data capture.

Change data capture is often used in a way where event driven architecture is masking what you’re actually doing, which is database replication.

That’s what I want to avoid.

If you’re using a CDC tool, that’s fine. But I still don’t want to expose the internals of the database directly to consumers.

Have some type of translation.

Take those internal database changes and translate them into an integration event. Turn them into something public that has an actual contract.

Maybe that event is more of a summary that can then be published to your broker and consumed for reporting.

You can build up a reporting data source from those summary events coming from different services and different sources.

Whether those events originated from CDC or something else doesn’t really matter.

What I want to avoid is using CDC to directly publish data model changes everywhere.

At that point, you’re just leaking internals.

Have a translation into something public and treat it like a contract.

## What Does the Consumer Actually Need This For?

There’s a really simple question you can ask yourself when designing events:

What does the consumer actually need this for?

If the answer is that it needs to react and do something when that event occurs, that’s a good answer.

I need to send an email when the shipment is delivered. Great.

The event isn’t that a status changed. The event is that the shipment was delivered. Now the event is explicit and the consumer knows exactly what happened.

If the answer is that the consumer needs a copy of this data as a local cache because it wants that data over there, that’s where I start asking more questions.

I’m not saying that’s automatically wrong. But it’s a smell to me. Do I really need that data over there? What functionality exists in that service that requires it? Should that functionality actually live there? Why do I need a local cached copy?

It’s not necessarily wrong. But it might be a sign that your boundaries aren’t aligned the way you think they are or that some functionality lives in the wrong place.

## Event Driven Does Not Mean Decoupled

If you’re using event driven architecture and asynchronous messaging, you need to be clear about what you’re actually doing.

You’re not necessarily decoupled just because you removed direct synchronous calls.

Sure, you removed the temporal aspect of the coupling.

But if you’re just distributing data to different services and all those services understand the internal state and data models of each other, you’re still coupled.

Now you’re coupled through messages.

If what you really want to do is distribute data around via events, that’s fine. Just call it what it is.

You’re distributing data around via events.

But if you’re using events as notifications so that other systems can react when important business concepts happen, that’s different.

The important part isn’t that you have a message broker or that everything is asynchronous.

It’s what the events actually mean.

	**Join CodeOpinon!**

	Developer-level members of my Patreon or YouTube channel get access to a private Discord server to chat with other developers about Software Architecture and Design and access to source code for any working demo application I post on my blog or YouTube. Check out my Patreon or YouTube Membership for more info.
