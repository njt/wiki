---
url: https://elektric.ai/blog/why-we-built-elektric-on-cloudflare
date_fetched: 2026-10-24
---

When we started building Elektric, we knew the hard part would not simply be calling an AI model.

Elektric sits between an application and a constantly changing ecosystem of models and providers. A single request can involve authentication, classification, routing, context, provider selection, fallback, streaming, usage tracking, billing, and telemetry before the answer ever reaches the developer.

We wanted an infrastructure stack that could support all of those pieces without turning Elektric into a collection of loosely connected services that we would have to operate independently.

That led us to Cloudflare.

Today, most of Elektric’s server-side application logic runs through a Cloudflare Worker. Around it, we use Cloudflare services for relational data, object storage, AI classification, provider transport, Knowledge retrieval, stateful coordination, asynchronous jobs, browser tools, email, and scheduled maintenance.

Cloudflare does not provide Elektric’s model routing or decide which model should answer a request. That logic is ours. What Cloudflare gives us is a set of infrastructure primitives that let us build those systems within one coherent platform.

## Workers sit at the center

The main Elektric service is a Cloudflare Worker called `elektric-api`.

It handles both our public API and application endpoints. The same deployment also serves the Elektric website, documentation, blog, and application frontend through Workers Static Assets. A separate Worker handles the `www`redirect.

More importantly, Workers is where the core AI request lifecycle comes together.

When an ordinary chat request reaches Elektric, the Worker can authenticate the project, validate the request, apply project limits, classify the work, select a routing policy, prepare Context, check model compatibility, consult health and billing information, acquire provider capacity, call the selected model, normalize the response, and record the result.

That is a lot of responsibility for one request path, but it is also exactly why the architecture works well for us. Instead of deploying a separate routing service, authentication service, context service, streaming gateway, provider proxy, and telemetry service, the orchestration can live inside one application runtime.

The answering model still runs at OpenAI, Anthropic, or Google. Other systems such as Stripe and Google OAuth remain external as well. Cloudflare does not somehow make the entire outside world disappear. But most of the logic that turns an application request into an AI request lives together inside Elektric’s Worker.

## We use AI inside the infrastructure itself

One of the more interesting parts of the architecture is that models are not only used to produce the final answer.

Elektric also uses Cloudflare Workers AI internally.

For ordinary text routing, a Qwen model running through Workers AI helps classify a request by category and difficulty. A separate classification determines whether private Knowledge may be relevant. The classifier returns structured metadata to Elektric; it does not answer the customer’s prompt.

That distinction is important.

Suppose a developer sends:

“Help me diagnose a concurrency bug in this payment service.”

The internal model may classify that as hard coding work. Elektric then uses that classification to determine which part of its routing policy applies.

The model performing the classification and the model eventually answering the request are two different layers.

We also use Workers AI in parts of Knowledge retrieval, including rewriting retrieval queries, and for document conversion in supported paths. This means AI itself becomes an infrastructure primitive inside Elektric, not merely the destination at the end of the request.

## AI Gateway connects us to the model providers

Once Elektric selects an answering model, provider traffic generally goes through Cloudflare AI Gateway.

Our OpenAI, Anthropic, and Google adapters each translate Elektric’s internal request format into the provider-specific format required upstream. Their responses are then normalized back into one interface for the application.

AI Gateway gives us a common transport layer for those provider calls, including authentication and correlation metadata.

But the distinction between AI Gateway and Elektric is important: **Cloudflare transports the request. Elektric decides where it should go.**

The classification system, candidate pools, model eligibility rules, health checks, and fallback decisions are implemented by Elektric. We are not delegating our routing policy to AI Gateway.

That separation is useful because provider transport and model selection are different problems.

Gateway gives us one place through which we can communicate with several model providers. Elektric then builds its own decision layer on top.

## Different data belongs in different places

AI infrastructure produces many different kinds of data, and forcing all of it into one storage system would make very little sense.

Elektric primarily uses D1 for structured application and operational state.

Our application database stores things like accounts, projects, API keys, billing information, Conversation, Memory, Knowledge metadata, files, jobs, and other application records. Separate D1 databases are used for telemetry and an independent deletion ledger. A Model Lab database is also configured separately.

Telemetry is particularly important for routing.

Elektric records requests and individual provider attempts separately. That means one customer request can be connected to its selected model, actual execution model, fallback behavior, latency, token usage, cost calculations, and failure information.

That request-to-attempt relationship matters because one Elektric request can involve more than one provider attempt.

R2 handles a different kind of data.

We use R2 for file bodies and other object-like data, including temporary uploads, generated assets, asynchronous job payloads, blog images, and benchmark evidence. Our Model Lab artifacts can also be preserved there, including corpora, candidate answers, judging results, and reports.

The distinction is simple: D1 is useful for structured state we need to query and relate. R2 is useful for larger durable objects.

We do not currently use Workers KV in the application.

## Knowledge is its own infrastructure system

When a project enables Knowledge, the architecture expands again.

Elektric stores document and upload metadata in D1 and sends the material into Cloudflare AI Search for managed ingestion and retrieval. When the application needs relevant Knowledge, Elektric can rewrite the query using Workers AI, search the scoped Knowledge index, select the relevant chunks, and add that information to the model’s context before execution.

We deliberately keep Conversation, Memory, and Knowledge separate.

Conversation provides recent thread history.

Memory provides durable information about a user.

Knowledge retrieves relevant information from organizational or project material.

Those systems can all contribute to the context given to the answering model, but they represent different kinds of information and have different persistence and retrieval behavior.

That separation matters because the model underneath the application can change while the application’s context remains stable.

## Durable Objects handle the state that should not be stateless

Workers are a natural fit for request handling, but some infrastructure problems require coordination across requests.

That is where we use Durable Objects.

Elektric currently uses them for three distinct jobs: per-project admission and limits, provider-capacity coordination, and realtime sessions.

Provider capacity is a good example.

Before executing a model request, Elektric can coordinate capacity for the relevant provider and model family. Repeated failures can affect the state maintained by that coordination layer, while successful execution can clear previous failure state.

This works alongside, rather than replacing, our D1-based health system.

D1 gives us historical operational evidence. Durable Objects give us a place for coordinated state around active requests.

## Streaming changes the architecture

AI responses are not ordinary HTTP responses.

Developers expect streaming, and different model providers represent streaming events differently.

Elektric’s provider adapters normalize those different streams into a common internal event format before the Worker exposes them through the public API.

Streaming also changes what fallback can do.

If the initial execution fails early enough, Elektric can attempt an eligible fallback. Once the prepared response stream has been committed, however, Elektric does not switch models in the middle of an answer.

That sounds like a small implementation detail. In practice, it is one of many examples of why AI infrastructure needs logic above a simple provider proxy.

## Cloudflare also handles the less glamorous parts

AI infrastructure is mostly discussed in terms of models, but production applications contain a large amount of work that has nothing glamorous about it.

Elektric uses Cloudflare Workflows for asynchronous provider jobs such as video operations. Cron Triggers run maintenance every five minutes, including cleanup, job recovery, billing reconciliation, notifications, and monitoring. Browser Run supports rendered-page retrieval for research tools. Cloudflare’s Email Service handles authentication emails, blog subscriptions, and notifications.

This matters more than it sounds.

The attraction of the platform is not any single one of these products. It is that we can compose them around the same Worker application without creating a new infrastructure stack for every feature we add.

## The economics make experimentation easy

There was another practical reason Cloudflare worked well for us: it is remarkably inexpensive to start building on.

Early in a product’s life, infrastructure usage is unpredictable. You are constantly adding things, testing ideas, throwing some of them away, and discovering that the innocent-looking feature you built on Tuesday now needs a database, object storage, background jobs, scheduled work, and an AI model by Friday. We wanted to be able to experiment without every new primitive immediately creating another meaningful fixed cost.

Cloudflare’s pricing model makes that unusually easy. Workers can be used on the Free plan with up to 100,000 requests per day, while the Workers Paid plan starts at $5 per month. The paid plan also includes initial usage across Workers and several related platform services, and Cloudflare does not add separate data-transfer or bandwidth charges to Workers.

The same pattern extends to much of the stack we use. D1’s Free plan includes 5 million rows read and 100,000 rows written per day, along with 5 GB of storage. On the paid Workers plan, the included monthly allocation rises to 25 billion rows read and 50 million rows written before usage charges begin. D1 also scales to zero, so there is no database compute capacity sitting around generating a bill when nothing is querying it.

R2 gives us another example. The free tier includes 10 GB-month of storage, one million Class A operations, and 10 million Class B operations each month. Standard storage beyond that is currently $0.015 per GB-month, and R2 does not charge for data transferred out to the Internet.

That last part is particularly attractive for infrastructure that moves files and generated assets around. Object storage on other large cloud platforms can include separate data-transfer charges. Amazon S3, for example, currently gives customers the first 100 GB per month of Internet data transfer out across AWS services for free and then charges for additional outbound transfer depending on the region and usage. R2’s pricing simply lists Internet egress as free.

The AI pieces are similarly approachable. AI Gateway’s core features are currently free, including its core analytics, caching, and rate-limiting capabilities. Workers AI includes 10,000 Neurons of inference per day at no charge on both Free and Paid Workers plans, with usage beyond that on the paid plan billed according to model consumption.

What stood out to us was the economics of the whole system.

We could start with a Worker, add relational storage with D1, object storage with R2, internal inference with Workers AI, provider transport through AI Gateway, then layer on Durable Objects, Workflows, scheduled jobs, and other primitives as the product became more sophisticated. Many of those services either have meaningful included usage or only begin charging materially as the application starts using them.

That matters for a company like Elektric because experimentation is part of building the infrastructure. We can try a new internal classifier, add another telemetry table, preserve benchmark evidence in R2, introduce a background workflow, or build a new context capability without first deciding whether the experiment deserves its own server, database cluster, networking layer, or monthly infrastructure commitment.

As the system grows, usage obviously stops being free. But the cost curve starts close to zero and rises with the product rather than forcing us to provision a large infrastructure footprint in advance.

For us, that has been one of Cloudflare’s less glamorous but most useful properties: **it is cheap to try things.**

And when you are still figuring out what the right AI infrastructure should look like, being able to try a lot of things is worth quite a bit.

## The Model Lab uses the same foundation

The routing system only becomes useful if we have evidence about the models underneath it.

That is the purpose of Elektric’s Model Lab.

Our evaluation system can run candidate models through the same provider adapter infrastructure, collect responses and costs, judge results using task-specific benchmarks, and preserve evaluation evidence. Some benchmark artifacts are stored in R2, while Model Lab also has D1 persistence implemented for structured evaluation data.

The important architectural separation is that evaluation does not automatically change production routing.

Benchmarks provide evidence.

Elektric decides how that evidence should change routing policy.

That gives us the ability to compare new models against the models already being used underneath an application and update the policy when the tradeoff improves.

## Cloudflare does not build Elektric for us

This is probably the most important distinction in the architecture.

Cloudflare provides many of the primitives, but it does not provide the system Elektric is building.

Cloudflare does not define our six routing categories. It does not decide whether a request is easy or hard. It does not maintain our routing tables. It does not define which models belong in which pools. It does not normalize every provider into our public interface. It does not decide when a fallback should happen. It does not design our benchmarks or determine how Context should be assembled.

Those systems are Elektric.

Cloudflare gives us compute, storage, model infrastructure, provider transport, stateful coordination, background execution, and other building blocks.

That is the relationship we wanted.

We did not want an AI platform that made every architectural decision for us. We wanted infrastructure that gave us strong primitives while leaving us free to build the intelligence layer ourselves.

## Why the architecture fits Elektric

Elektric’s job is ultimately one of coordination.

A request arrives from an application. We need to understand it, decide which model should handle it, assemble the right context, communicate with different providers, recover from failures when possible, normalize the answer, account for what happened, and keep the developer-facing interface stable while everything underneath continues to change.

Cloudflare gives us a platform where many of those pieces can live together.

Workers gives us the main execution environment. Workers AI gives us models we can use inside the infrastructure itself. AI Gateway gives us a common provider transport. D1 gives us structured application and operational state. R2 gives us object storage. AI Search supports Knowledge. Durable Objects give us coordinated state where we need it. Workflows and Cron handle work that should not live inside the synchronous request.

None of those products alone is the reason we built Elektric on Cloudflare.

The reason is how they fit together.

We can keep the application interface simple while building a fairly sophisticated system underneath it, and we can add new infrastructure capabilities without having to create an entirely new operational stack every time.

For a company whose goal is to hide the complexity of AI infrastructure from developers, that is a useful property for our own infrastructure to have.
