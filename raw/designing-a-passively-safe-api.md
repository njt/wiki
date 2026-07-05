---
url: https://www.danealbaugh.com/articles/passively-safe-apis
date_fetched: 2026-07-05
backfilled: true
---

Follow me at @dane_albaugh

I'm in the process of migrating Augno's monolithic API to a microservices architecture. It's been slow, largely because we're making every public endpoint passively safe.

A *passively safe* system is one that is designed to fail gracefully. Crumple zones in cars, seismic zones in buildings, and gravity-driven cooling systems in nuclear reactors are all examples of passively safe designs.

In APIs, **passively safe** means failures (crashes, timeouts, retries, partial outages) can't produce duplicate work, surprise side effects, or unrecoverable state. After any failure, the system either (a) completes the workflow exactly once, or (b) lands in a terminal, explicitly visible state that won't double-bill or duplicate work.

Consider an endpoint that has the following characteristics:

- It must call an external API in-band and cause side effects.
- It must perform asynchronous work in response to the request.
- It must create a new resource and update several related resources across multiple services.
- Clients must be able to retry without creating duplicates or extra charges.

Any failure at the wrong moment could leave the system in an unrecoverable state. So, how can we make such an endpoint passively safe? Let's think through a deliberately gnarly example and see if we can come up with a solution.

### An illustrative example

Imagine an API endpoint that allows users to ship an order of goods: `POST /shipments`. There are many things that must happen to create this shipment.

- A third-party API must validate the shipping address.
- A third-party API must generate tracking information and labels.
- The shipment and shipping cases records must be updated with tracking information.
- An invoice record must be generated against the shipment record.
- The order record must be either marked as fulfilled or partially fulfilled.
- The customer, sales representative, and other interested parties must be notified of the shipment.

The most straightforward approach would be to implement this endpoint in a monolith. Each step would be performed synchronously and any encountered error would be sent directly to the user. Assuming all goes well, the request would look something like this:

#### Issues

If you think through the failure modes this endpoint might hit, a few issues jump out. Here are the big ones:

##### 1) External API calls cannot occur in a transaction

Because we use an ACID-compliant relational database, we can ensure that changes to our own resources either succeed or fail together. Since this is a monolith, there is only one transaction needed to update all the local data pertinent to the request. But we have no way to roll back changes that happen outside our local transaction boundary, whether that's third-party APIs or other services we own. Moving forward, we'll call these *foreign state mutations*.1

If the server fails during the transaction, we'd end up with orphaned shipping labels because the external API generated them before the transaction began.

##### 2) Requests are not retry safe

If a user encounters an unexpected error, they have no way to know whether it's safe to retry. If the request failed during the transaction, retrying might bill the user again for orphaned shipping labels. If the request failed while sending the notification, retrying would create a duplicate shipment record.

##### 3) External API outages = downtime

Any failure in an external API dependency will cause an outage in this endpoint, which creates frustrating user experiences.

##### 4) Synchronous = slow

Since the request is entirely synchronous, it can take 2-30 seconds. That further increases the risk that the client disconnects or the request times out.

#### Target properties

Before we start fixing anything, here's the checklist we're aiming for:

- No external side effects inside DB transactions
- Durable checkpoints for recovery
- At-least-once delivery + dedupe
- Idempotent request handling

#### Making some tasks asynchronous

A concrete next step is to introduce a message broker. A message broker is an intermediary that asynchronously routes, buffers, and reliably delivers messages between services. After some consideration, we decide it isn't necessary to notify the customer synchronously. It's fine if they receive an email a few minutes after the request completes.

We set up a background worker that listens for `notification.cmd.send_email` messages and processes them out of band. With this new approach, our request now looks something like this:

Although we have improved the user experience by providing a faster response, we have not solved any of the issues we originally had. If you look closely, we have actually introduced some new issues!

##### 5) Message delivery is not guaranteed

If a server error were encountered after the transaction, the message might never be added to the queue. Some customers may never receive their invoice or shipment notifications, leading to late payments and unhappy customers.

##### 6) Messages may be delivered multiple times

If the dequeue operation fails, the message broker will attempt to deliver the message again. This could lead to duplicate invoice payments and a major headache for our accounts receivable team.

#### Guarantee messages are delivered at least once

We can address a few of these issues by implementing transactionally staged jobs2 aka a message outbox.3 The core idea is to add a table to the database `message_outbox`. We will insert rows into this table for each message we want to send. A background worker, which we will call the enqueuer, will periodically drain this table, publish each row to the message broker, and delete it only after the broker acknowledges receipt.

With this new approach, our request now looks something like this:

You will notice a few changes to our design. We now have an `insert_outbox()` inside the transaction guaranteeing that if our transaction rolls back, we will never commit the message to the outbox. Next, we have a separate enqueuer background worker which will periodically drain the `message_outbox` table and send messages to the message broker. The enqueuer only deletes the message from the `message_outbox` when the message broker confirms receipt (so the outbox gives **at-least-once publish**).

This approach has some powerful benefits.

- Since messages are only sent if the transaction commits, we will never create orphaned messages on rollback.
- Messages are guaranteed to be delivered at least once since we only delete from the outbox after they are successfully published.
- We can send the user a response as soon as the transaction commits, without waiting for downstream processing.

#### Guarantee messages will be processed at most once

Now that we can guarantee messages will be delivered at least once, we should assume we may receive duplicates. To address that, we can use a message inbox. The idea is similar to our `message_outbox` table, but in reverse. We create a new table called `message_inbox`. This table has a couple of important columns: `status` and `message_id`. Status is either `received` or `processed`, and `message_id` is a unique ID for the message we saved in our outbox. We also have a `failed_at` column to record a failed attempt at processing a message.

When a background worker receives a message, it first attempts to insert `(message_id, status='received')` into `message_inbox`. If the insert fails on the unique constraint, the message is a duplicate and we can handle it based on its current `status` and `failed_at` values. If the insert succeeds, the worker performs the work and then updates the row to `status='processed'` (or sets `failed_at=now()` on failure) **before** acknowledging/dequeueing the broker message.

Now, our request looks something like this:

This design lets us de-dupe redeliveries and converge on a single visible outcome per `message_id`. Consider a few scenarios:

##### Duplicate message received

The broker redelivers after we've already processed the message. The inbox insert hits the unique constraint and we see `status='processed'`, so we can acknowledge and drop it.

##### Background worker fails in a way we expect

A prior attempt set `failed_at`. On redelivery we see `failed_at != NULL` and can choose to retry (if safe) or leave it unprocessed / dead-letter it.

##### Background worker fails in a way we did not expect

A crash occurs after inserting `received` but before setting `processed`/`failed_at`. On redelivery we see `status='received'` and can retry, or treat it as “in-flight” until a lease/timeout expires.

##### Background worker attempts to process a message currently being processed

Two workers race: one wins the insert, the other hits the unique constraint and sees `status='received'`. To prevent double side effects, use a lease/lock (e.g., `processing_started_at` + `processing_owner`) and only allow a worker with the active lease to perform side effects.

#### Making the request idempotent

Now that we have at-least-once delivery plus a de-dupe mechanism, let's work on making the request safe to retry.

##### What is idempotency?

Idempotent requests are those that can be made multiple times without causing unintended side effects. In other words, submitting the same request multiple times should have the same effect as submitting it once.

###### GET, PUT, DELETE

The RFC for HTTP semantics4 specifies that the `GET`, `PUT` and `DELETE` methods are idempotent *by definition*, as are safe requests.

| Method | Idempotent | Notes | 
|---|---|---|
| `GET` | Yes | Reading data multiple times has no intended side effects | 
| `POST` | No | Creating/performing actions multiple times can have side effects | 
| `PATCH` | No | Not idempotent by definition and potential design hazards (e.g. race conditions) 56 | 
| `PUT` | Yes | Full updates are idempotent by definition | 
| `DELETE` | Yes | Deleting multiple times has same effect as deleting once | 


Note: Since users will assume`GET`,`PUT`and`DELETE`requests are idempotent, you must be careful to ensure that you handle these requests with this expectation in mind. When designing endpoints, you should follow these semantics: use`POST`for operations that are not idempotent,`PUT`for mutations that are idempotent, and`PATCH`for mutations that are not idempotent.7

###### POST

`POST` requests are not idempotent. Consider the following `POST` request:

```
POST /messages
Content-Type: application/json
{
    "message": "Hello, world!"
}
```
```
POST /messages
Content-Type: application/json
{
    "message": "Hello, world!"
}
```
If this request fails mid-flight, the user can't know whether it's safe to resubmit. What if the message was created but the response never made it back to the client (timeout, disconnect, crash)? Retrying might create a duplicate. This is true of any request that creates new records, which is why `POST` is not considered idempotent by default.

###### PATCH

Similarly, `PATCH` requests are not idempotent. Consider the following request:

```
PATCH /account/123
Content-Type: application/json
{
  "op": "increment",
  "field": "balance",
  "value": 10
}
```
```
PATCH /account/123
Content-Type: application/json
{
  "op": "increment",
  "field": "balance",
  "value": 10
}
```
If this request were to fail, your user could not resubmit it safely. Doing so might inadvertently increment the account balance twice.

##### Idempotency keys

If we wish to make `POST` and `PATCH` requests idempotent, we must come up with a way to identify each request attempt uniquely. This is where idempotency keys come in.

An idempotency key is a unique identifier that a client generates to identify a particular request attempt. The client sends this key to the server via the `Idempotency-Key` header.8 Once the server has received the request, it will immediately store the idempotency key and begin processing the request. As the request progresses, the server will note the state of the request and store the latest recovery point with the idempotency key. When the request is complete, the server will update the status of the idempotency key and save the response.

```
POST /messages
Idempotency-Key: 123e4567-e89b-12d3-a456-426614174000
{
   "message": "Hello, world!"
}
```
```
POST /messages
Idempotency-Key: 123e4567-e89b-12d3-a456-426614174000
{
   "message": "Hello, world!"
}
```
If the first attempt to submit the request fails, the client can retry the request using the same idempotency key. When the server receives a repeated idempotency key, it recognizes the request and can determine the current state of the request. Depending on the situation, the server can:

- Resume processing from the last saved recovery point if it is safe to do so
- Abandon the request if retrying might cause problems
- Simply return the cached response from the original attempt

##### Some key terms

Although the concept of idempotency keys is simple enough, it is a bit tricky to implement. Each request that should be made idempotent via idempotency keys must be organized into recoverable phases. To help explain this planning process, we should define a few terms by which we can describe our endpoint phases:1

- **Foreign state mutation**: A mutation to any state outside the local transaction boundary (including other services you own and third-party APIs).
- **Atomic phase**: A set of- *local*state mutations that occur- *in a transaction*between foreign state mutations.
- **Recovery point**: A checkpoint that we have to get to after having successfully executed any atomic phase or foreign state mutation (note: these should be committed as part of their phase's transaction).
- **Final failure**: A failure that cannot be retried.

##### Breaking down the request lifecycle

Now that we understand the concepts, let's break down our request into the phases of its lifecycle.


Note: Ideally, all foreign state mutations would be moved to background jobs where`message_outbox`/`message_inbox`can ensure they are processed effectively. For the sake of this example, we will assume that address validation and shipping label generation must be handled in-band.

###### Grouping rules

Before we outline the atomic phases in our particular endpoint, here are some practical grouping rules (quoting directly from Brandur here1):

- Upserting the idempotency key record gets its own atomic phase.
- Every foreign state mutation gets its own atomic phase.
- After those phases have been identified, all other operations between them are grouped into atomic phases.

###### Tasks

Let's take a look at our new task list:

- **The idempotency key record must be created.**
- A third-party API must validate the shipping address.
- A third-party API must generate tracking information and labels.
- The shipment and shipping cases records must be updated with tracking information.
- An invoice record must be generated against the shipment record.
- The order record must be either marked as fulfilled or partially fulfilled.
- The customer, sales representative, and other interested parties must be notified of the shipment.
- **The idempotency key record must be updated with the status of the request and the server's response.**

###### Atomic phases

First, let's identify the foreign state mutations in our endpoint. We call two external APIs: a shipping API that's used to validate an address and generate shipping labels, and a notification API that sends notifications to our users. Since we have implemented the message outbox pattern, we no longer directly invoke the notification API.

Here are the atomic phases and their recovery points:

- Create the idempotency key - `started`
- Validate address; persist `address_validated`recovery point -`address_validated`
- Generate shipping labels; persist `tracking_generated`recovery point -`tracking_generated`
- Generate an invoice and update the order status; enqueue an update event - `update_event_sent`
- Finalize response and enqueue notification event(s) (outbox) - `completed`

Technically, validating the address is not a foreign state mutation since we don't mutate any state in the shipping API. However, we generally want to avoid executing any network request inside a database transaction, so we move this into its own phase.

###### The `tracking_generated` Atomic Phase

`tracking_generated` Atomic PhaseEach atomic phase will be executed in a single transaction between foreign state mutations. We will save recovery points as we progress through the request as part of their phase's transaction. To better illustrate what this looks like for each phase, let's zoom in on the `tracking_generated` phase.

In this phase, we start by requesting that the shipping API create new shipping labels and tracking information for our shipment. After we receive this information, we then open a transaction and save the tracking information to our shipment and update the recovery point.

If our request were to fail mid-way through this atomic phase, could we safely retry it? It depends. If our shipping API supports idempotency keys, we can safely retry the request. If they don't support idempotency keys, we might be able to make some query against their API to see if the labels were created in the previous request and, if so, retrieve them. If that is not an option, we might decide that this request is not recoverable and send a definitive error to the client, updating the idempotency key so that subsequent retries are short-circuited with our new cached error response. Regardless of our choice, this discrete phase can now handle retries gracefully.

##### Back to our example

We will create a new table called `idempotency_key` to store the idempotency keys for each request. This table should have a unique constraint so that an idempotency key can only be used once per route, method, and user. We will save the `recovery_point` to `started`.

The user will now submit their `POST /shipments` request to the server with the `Idempotency-Key` header and a unique key for that request. The server will immediately note the presence of the `Idempotency-Key` and insert a record into the `idempotency_key` table and the `status` is set to `received` by default. After we insert this row, we process the request and complete the work. As we make progress, we will update the `idempotency_key` with the latest recovery point value. If the request succeeds, we set the `recovery_point` as `completed` and save the response for replay. Then, we return our response to the user.


Note:You might notice that the idempotency key pattern is similar to our`message_inbox`pattern. The`message_inbox`table uses a unique`message_id`as its idempotency key and uses two recovery points:`received`and`processed`.

#### A passively safe design

Our endpoint is now passively safe and able to be retried safely. Consider a few failure modes:

##### The request fails during address validation

Because address validation is isolated into its own phase (outside any DB transaction), a failure here can't leave partially-committed local state behind. On retry with the same `Idempotency-Key`, the server sees the request is still in `started` and can safely re-run validation (or short-circuit with a cached “final failure” if the error is deterministic, like an invalid address).

##### The user's request never reaches the API server

If the request truly never reached the server, a retry with the same `Idempotency-Key` just behaves like the first attempt. If it did reach the server but the response was lost, the retry deterministically returns either `409 Conflict` (still processing) or the cached completed response, preventing duplicate shipments.

##### The user accidentally sends two requests at once

If both requests share the same `Idempotency-Key`, the unique constraint ensures only one execution “wins” and the other gets a deterministic response (in-progress conflict or replay). This converts a concurrency mistake into a safe, observable outcome instead of a double-charge / double-shipment incident.

##### Our shipping API is down

The in-band shipping phases fail fast, but the idempotency record preserves exactly how far we got (e.g. started vs address_validated) so callers can retry without redoing completed work. If the failure happens around label generation, we either resume using the shipping provider's idempotency support (best case) or mark a final failure to prevent repeated foreign mutations and surprise billing.

##### Our notifications API is down

The primary request can still succeed because notifications are decoupled behind the outbox and processed asynchronously. When the notifications service recovers, the enqueuer/worker pipeline drains the outbox and the inbox de-dupes any retries, yielding “delayed but correct” instead of “down or duplicated.”

##### An unexpected breaking change is introduced in the shipping API after an upgrade

A breaking change becomes a deterministic failure in a single, well-defined phase, rather than corrupting multiple internal tables mid-flight. Once identified, requests can be retried for affected idempotency keys (preventing repeated foreign mutations) while rolling out a code fix or version pin.

##### The server crashes during any atomic phase

Every atomic phase either commits fully (including its recovery point) or rolls back, so a crash can't strand half-applied local mutations. After restart, retries with the same `Idempotency-Key` resume from the last committed recovery point, while the outbox/inbox pair ensures downstream messages are neither lost nor applied twice.

### Implementation notes

Here are some final notes on how to implement idempotency in your API.

#### What should idempotency keys look like?

Idempotency keys will be indexed, so it can make sense to require UUIDs for `Idempotency-Key` to keep indexing efficient.9 It's also a good idea to validate keys against a published format to prevent abuse and to avoid embedding sensitive information in the value.8

#### The `is_transient` flag

`is_transient` flagErrors often fall into two buckets: deterministic (e.g. invalid input) and transient (e.g. conflicts, rate limits, intermittent 5xx). Instead of encoding this policy implicitly by status code, you can include an explicit boolean `is_transient` in your standard error envelope and have the idempotency layer consult only that field. If `is_transient` is `false`, cache and reuse the error response for the same `Idempotency-Key`. If `true`, do not cache so callers can retry. This delegates classification to the server on a per-error basis (you can override defaults when context requires), simplifies clients that can now key retry decisions off a single field and decouples behavior from any particular set of status codes.10

#### Retry scheduling and the thundering herd problem

It is worth noting that there are some considerations when retrying requests that have failed. The client should use exponential backoff with jitter to prevent overwhelming the server with retries.11 For example, Stripe's client libraries combine idempotency keys with a polite retry strategy. They decide **whether** to retry using signals like the `Stripe-Should-Retry` header and **when** to retry using `Retry-After` when present or an exponential backoff with jitter when it's not.12 13 This ensures retries are safe, avoid duplicating work and prevent "thundering herd" effects where many clients retry at the same instant.11 AWS recommends coupling capped exponential backoff with randomized jitter so clients use exponentially increasing delays up to a maximum cap and jitter spreads those retries across time.14 Without jitter, retries can sync up into concentrated spikes that worsen system stress. Jitter breaks that alignment and helps stabilize recovery.14 You should avoid using a timestamp for the `RateLimit-Reset` header as clients may all simultaneously retry at exactly the same time, creating the thundering herd problem.15 Instead, use seconds like `Retry-After`.

#### Idempotent responses

For an idempotent `POST` request, the first successful call returns `201 Created`. If the request is retried, the server should return `200 OK` and the same resource representation.8 16 If a request with the same `Idempotency-Key` is still being processed, the server should return `409 Conflict`.9 (Some APIs use `202 Accepted` + polling instead; `409` is one reasonable choice when you want to communicate “still processing.”) Some errors are transient (e.g. `429 Too Many Requests` and `503 Service Unavailable`). Their results will not be stored.9 For `DELETE` requests, the server should return `204 No Content` for the first request and a `410 Gone` for subsequent retries.17 If the record does not exist, the server should return `404 Not Found`. If a stored response is returned, the server may provide an `Idempotent-Replay: true` header.18 9

#### Should idempotent retries succeed after a failed first request?

In the first version of Stripe's API, retries returned the previously-saved response from the first request, even if it was an error.19 In V2, they retry some failed requests when doing so can't cause side effects, and then return an updated response.19 In many cases, that better matches user expectations. If an internal server error caused the first attempt to fail, it's usually better to try again once the incident is resolved.

#### Should you allow the user to retry the request with a new body?

If a user resubmits a request with a different body but the same idempotency key, it may be tempting to allow it. In practice, this encourages "retry by mutation" and can create surprising side effects. Instead, hash the request body and only allow a retry with the same idempotency key if the hash matches.

#### How long should you respect the `Idempotency-Key`?

`Idempotency-Key`?In the first version of Stripe's API, they considered two requests idempotent if they occur within 24 hours of each other.19 In V2, they have increased that to 30 days. I haven't been able to find commentary on this change. Brandur makes an offhanded comment that he believes the selection of 24 hours was arbitrary here.17 Make a cutoff that makes sense for your particular application and user requirements.

#### How do we handle abandoned or failed requests?1

1

If you rely exclusively on clients to retry indeterminate requests (timeouts, disconnects, “did it go through?”), you'll eventually accumulate keys stuck in a non-terminal recovery point because some clients never come back. A practical fix is to run a small “completer” process that periodically scans for idempotency keys that are old enough to be suspicious but not old enough to be deleted, then re-drives them forward using the same idempotency machinery (i.e., resume from the last committed recovery point, respecting the phase state machine).

Implementation details that tend to matter in practice:

- **Eligibility rules**: only pick keys that are- `received`/- `in_progress`, whose- `updated_at`is older than some threshold, and that haven't exceeded a max attempt count.
- **Safety rails**: rate-limit the completer, add exponential backoff, and stop retrying on deterministic errors (treat as terminal).
- **Operational visibility**: when a key has been “stuck” too long (or hit max attempts), move it to a quarantine list/table for manual inspection rather than letting it loop forever.

#### How can we clean up old idempotency keys?1

1

Idempotency keys are a correctness mechanism, not a permanent request archive, so you want a TTL and a “reaper” process that deletes keys once they're no longer needed for safe retries. Brandur suggests a ~72-hour threshold as a reasonable default so you can still recover from weekend incidents and let a completer finish stragglers after a fix ships.

A reaper is usually simplest when you make the lifecycle explicit:

- **Only reap terminal keys**(e.g., completed or “final failure”) older than your retention window.
- For **non-terminal keys**older than the window, try one last completion/cleanup pass (or quarantine them) before deletion, so you don't silently lose evidence of a stuck workflow. This mirrors Brandur's suggestion that an ideal reaper notices requests that couldn't be finished and escalates them for humans.
- **Delete in small batches**to avoid table bloat/lock contention; if volume is high, consider time-partitioning so reaping becomes “drop an old partition” instead of row-by-row deletes.

If you still want long-term observability, keep a separate, compact “request ledger” (e.g., key hash, route, timestamps, terminal outcome) and delete the heavyweight replay payload/state. That preserves metrics/audit value without keeping full idempotency state indefinitely.
