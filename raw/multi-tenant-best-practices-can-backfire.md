---
url: https://codeopinion.com/multi-tenant-best-practices-can-backfire/
date_fetched: 2026-10-03
---

**Sponsor:** Do you build complex software systems? See how NServiceBus makes it easier to design, build, and manage software systems that use message queues to achieve loose coupling. Get started for free.

You have a multi-tenant system, but you discover a pretty horrible bug. How can that be? You followed all the best practices!

Your HTTP API places a message on a queue to generate an invoice. A background process picks up that message and executes it. Everything looks like it worked. There were no exceptions, no failed database calls, and no errors in your logs.

However, something went horribly wrong.

The request was supposed to execute for tenant A, but what actually happened is that it executed for tenant B. The wrong tenant.

## YouTube

Check out my YouTube channel, where I post all kinds of content on Software Architecture & Design, including this video showing everything in this post.

You likely tried to prevent this from ever happening in the first place. Maybe you made the design decision that you didn’t want the tenant ID littered everywhere and passed around from your HTTP API, to messages, to background tasks, to your database.

Instead, you preferred to make it invisible. You establish some type of context and everything executing within that context has access to the tenant.

But here’s the problem.

The context of an HTTP request and the context of processing a message from a queue in a separate background process are completely different.

How did you build the tenant context from the message?

You might think creating that context solved the problem of passing the tenant ID around everywhere. All you really did was create a different problem.

And that’s the theme with multi-tenant systems.

## Best Practices Still Have a Cost

There are a lot of best practices and rules around multi-tenant systems.

Don’t pass the tenant ID around everywhere.

Use global query filters. Use a database per tenant. Use feature flags.

None of those recommendations are inherently bad. What they often leave out is the tradeoff and the cost.

Every time you identify a problem and come up with some type of solution, you’ve usually created another problem. That new problem then needs another solution. And this just keeps going.

Often, you feel some pain in your system and come up with a solution that seems like it will resolve that pain and remove complexity.

But you’re usually not removing complexity. You’re moving it.

So instead of asking, “What’s the right way to build a multi tenant system?” I think a better question is:

**What are the tradeoffs of the decisions I’m making, and where am I putting the complexity?**

## Explicit Tenant Identity

Let’s go back to passing the tenant ID everywhere.

Maybe as part of an HTTP request, the tenant is identified through some identity information. When you create a message, you include the tenant ID in that message. When a background process picks it up, the tenant ID is there. When you interact with your database, the tenant is part of that operation.

It’s everywhere. But the thing is, it’s explicit.

It’s obvious and visible in your HTTP requests, your messages, your logs, and your database operations.

A fair question is why are we so coupled to tenancy?

If multi tenancy is a core part of our system, passing the tenant ID everywhere can feel like nothing but noise.

And it is noise. But it’s visible noise. You know the dependency exists because you’re forced to deal with it.

## Implicit Tenant Context

Maybe you decide that passing the tenant ID around everywhere is a problem.

Instead, you establish the tenant once at the beginning of some process. Maybe you identify it from the host name. Maybe it comes from an API key. The details don’t really matter.

You identify the tenant once, establish some type of context, and everything inside that process understands which tenant it belongs to.

Now you don’t need the tenant ID flowing through every method call or request.

That can absolutely feel “cleaner”. But what did we really do? We didn’t remove tenant identity. We hid it.

That’s the tradeoff.

If tenant identity is explicit, you have a visible dependency. It has to travel with every operation. It’s repetitive and you notice it because the tenant ID is everywhere.

If tenant identity is implicit through some context, your APIs can be simpler. There’s less plumbing and you don’t need tenant IDs appearing in every method, message, or request.

But now you have a hidden dependency.

That context has to exist.

And when you move from one execution context to another, such as from an HTTP request to processing a message, you have to rebuild it.

That’s exactly how you can end up with the bug from the beginning.

The tenant context didn’t disappear just because you stopped passing the tenant ID around.

## Shared Database or Database Per Tenant?

Now let’s get into the topic everybody loves around multi-tenancy.

Should you have a shared database or a database per tenant?

I’m not talking about the infrastructure side of this. Infrastructure can give you different levels of isolation depending on what you need, but your design is equally important.

The biggest concern people have with a shared database, rightfully so, is leaking information from one tenant to another.

Tenant A makes a request to your API, which hits your database, and you need to be absolutely positive that you’re only returning data that belongs to tenant A.

Generally, you want this to happen automatically.

You don’t want every query you write to require you to remember to add another filter for the tenant.

There are different ways to accomplish this. Global query filters are one option. Row-level security is another. The specific mechanism isn’t really the point.

The goal is that filtering happens automatically.

Of course, a database per tenant gives you a different type of isolation. But there are tradeoffs there as well.

From a design perspective, there’s a meaningful difference between saying:

“Give me all orders.”

And:

“Give me all orders for this tenant.”

In many systems, you eventually need to intentionally cross tenant boundaries.

Maybe it’s for reporting. Maybe it’s administration. Maybe it’s analytics.

If you’re relying on global query filters or row level security, you might need some type of escape hatch that allows certain operations to cross those tenant boundaries.

Now you’ve created an exception to the rule.

That exception can be dangerous, but you still need it.

Again, you solved one problem and created another.

You introduced automatic filtering so developers wouldn’t need to think about tenant filtering on every query. That gave you a safer default.

But now you have the problem that sometimes you actually do need to cross tenant boundaries.

So you build an escape hatch.

With a database per tenant, there might not even be a boundary to cross in the same way. You have to query each database individually or aggregate the information somewhere else for reporting or analytics.

Different solution.

Different problem.

Different place for the complexity.

## Multi-Tenancy Is More Than Data

Multi-tenancy isn’t just about data. It’s often about functionality and features.

Maybe tenant A gets access to pricing and reports.

Tenant B gets reports plus some custom functionality developed specifically for them.

Tenant C gets pricing and approvals, but not reports.

Each tenant starts having its own features and workflows.

The first thought is usually configuration or feature flags. You define which features each tenant has access to, and you keep one application that changes its behavior based on that configuration.

At first, that can absolutely be the right approach. But eventually you might start asking another question.

Do we actually have multiple products inside one codebase?

Maybe you start turning these differences into modules. Tenants get access to specific modules. Those modules might have their own implementations, their own deployments, or even different versioning.

So what happened? We didn’t remove complexity. We moved it.

With feature flags and configuration, you have conditional complexity inside a single application.

With modules or separate implementations, you have structural complexity around organization, deployments, versioning, and boundaries.

Neither one magically removes the complexity of having different tenants that need different behavior.

It just determines where that complexity lives.

## The Pattern Is Always the Same

There’s a pattern across all of these examples.

If the tenant ID is everywhere, it’s explicit and visible, but it’s noisy and repetitive.

If you create a tenant context, it’s convenient and simpler to use, but it’s implicit.

If you use automatic filtering, you get a safer default, but you also introduce hidden behavior. And if you need an escape hatch, you now have an exception that you need to think about.

If tenant identity is scoped explicitly, you have clear ownership, but that context is going to show up everywhere.

If you use configuration and feature flags, you have one flexible application, but you can end up with conditional complexity everywhere.

If you move toward modules or separate implementations, you get more isolated behavior, but now you have structural complexity around how those modules are organized and deployed.

There’s one thing in common with all of them.

**You’re not removing complexity. You’re moving it.**

## What Are You Trading?

That’s why I despise the idea that there’s one right way to build a multi-tenant system.

“You should use a global query filter.”


“You shouldn’t use a global query filter.”


“You should pass the tenant ID through every part of the application.”


“You should never pass the tenant ID around.”


Statements like these ignore the important question.

**What are you trading?**

Instead of asking what pattern you should use or what the best practice is, ask a different set of questions.

Where does the tenant ID actually come from?

Where should tenant identity be explicit?

Where is it safe for tenant identity to be implicit?

What happens when you move from one execution context to another?

Where can operations intentionally cross tenant boundaries?

How are tenant-specific features represented?

Where are you willing to accept complexity?

Because when something is explicit, it’s visible. When it’s implicit, it’s convenient. It’s really no different from thinking about infrastructure.

Shared infrastructure can be efficient. One shared database is an obvious example.

A database per tenant can give you more isolation and control.

The same type of tradeoff exists in your application design.

Explicit gives you visibility. Implicit gives you convenience.

Neither removes complexity.

You’re deciding where you want that complexity to live.

	**Join CodeOpinon!**

	Developer-level members of my Patreon or YouTube channel get access to a private Discord server to chat with other developers about Software Architecture and Design and access to source code for any working demo application I post on my blog or YouTube. Check out my Patreon or YouTube Membership for more info.
