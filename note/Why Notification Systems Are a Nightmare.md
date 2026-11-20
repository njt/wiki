# Why Notification Systems Are a Nightmare

A step-by-step anatomy of how a one-line "send a receipt email" ticket grows, over three months, into a 25-line `notify()` function plus a workflow engine — with each step anchored to a real public incident (Salesloft, Zoho, Buttondown, HBO Max, Everbridge). Written by the author of the open-source notifkit, which makes it part engineering essay, part product plug.

---

The essay's structure is accretion: each fix creates the next problem. A provider timeout demands retries; retries double-send, so each message needs an ID stable across retries; but the "mark as sent" write must go *after* the send, which leaves an unavoidable crash window — so at-least-once delivery is a ceiling, not a bug. A provider outage demands failover, which leaks the ID (the backup never saw it). User comfort demands quiet hours held in a database (a `setTimeout` dies with the next deploy) and time zones stored as IANA names, not offsets. Bulk sends demand rate limiting with *randomised* backoff — deterministic backoff is exactly what caused Buttondown's duplicate newsletters — plus per-topic consent, blocked lists, bounce webhooks, and signed unsubscribe links. The most famous failures (HBO Max, Everbridge) aren't about recipient preference at all but about sender intent: a test reaching a real inbox. The fix is making that impossible — route all non-production traffic to a log-only provider. Finally, five messages per order fired from disconnected handlers force an order-level workflow that must wait for days, survive deploys, and stop on cancellation.

The closing move is honest: notifkit is the packaged version of all of it, MIT-licensed, self-hosted on your own Postgres and Redis, and it explicitly does *not* promise exactly-once delivery — because when Gmail reports failure for an email it actually delivered (the Salesloft incident), no client-side design can help. The real moral is that no single piece is hard; they all touch each other.

---

## Key quotes

> A timeout only means *you* didn't get an answer. It doesn't mean the provider didn't send the email.

The cleanest one-paragraph statement of why notification delivery is a distributed-systems problem wearing an application costume. Timeouts are ambiguous; the fix (provider-acknowledged idempotency IDs) must live partly outside your code.

> One thing in there is sneaky: the "sent" mark goes *after* the send. Write it before, and a crash in between means the receipt never goes out. Write it after, and a crash in between means the retry sends a second one. You don't get an option where neither can happen.

An unusually plain-language statement of the classic dual-write problem. "You don't get an option where neither can happen" is the whole thesis in one sentence.

> All of those checks ask whether Ana wants the message. None of them asks whether *you* meant to send it.

The sharpest observation in the piece: consent checks and blocked lists guard the recipient, but HBO Max and Everbridge failed on the sender side. Environment isolation (log-only provider outside production) beats every recipient-side check.

> None of these pieces is that hard on its own. The hard part is that they all touch each other.

The essay's actual conclusion, seconded by Sophie Alpert's quote about Slack's notification logic. Complexity here is not in the pieces but in the constraint coupling.

## Key themes

#concept #pattern #tool

- **Idempotency and at-least-once delivery** — stable message IDs, mark-after-send, and why exactly-once is unavailable when providers lie about failure.
- **Accretive complexity** — "simple" features that grow one support ticket at a time until the code is a workflow engine.
- **Sender-side failure** — the test-to-production leak as a category error no recipient check can catch.
- **Self-interested credibility** — the author sells the problem's solution; the piece mitigates this by being MIT-licensed and by insisting the pseudocode stands alone.

## Analysis

This is a product-marketing essay that earns its keep. The pseudocode is precise enough to be wrong in interesting ways (and the author flags the interesting ways himself), and the incident anchors give each step a receipt. The weakest link is the framing: "one line becomes a nightmare" is partly a rhetorical setup for notifkit, and the honest counter-reading is that a notification *service* is a legitimate product category — most teams should buy or adopt rather than build, which is the essay's point but also its sales pitch.

The deeper value is as a case study in hidden distributed-systems requirements leaking into apparently mundane application code. It pairs naturally with thinking on idempotency and messaging semantics elsewhere in the wiki.

This piece strengthens [[Idempotency Is Easy Until the Second Request Is Different]] with a worked, end-to-end example of exactly its thesis: the ID is easy, the mark-sent ordering is the trap, and the provider must cooperate for the fix to be complete. It nuances [[Ordering, Grouping and Consistency in Messaging Systems]] by showing what "consistency" means when the message is a human-facing email and the failure mode is Ana getting two receipts at 2:40 am. It complicates [[The Valley of Webhooks]] with the webhook-adjacent underside — bounce and complaint webhooks, signed unsubscribe tokens, and courier events you can't distinguish from fabrications. And it gives [[Queues Don't Fix Overload]] a concrete illustration of why queues alone aren't the answer: the queue arrives at step one (backgrounding the send) and by step seven you've built rate limiting, per-user caps, a scheduler for held messages, and a workflow engine on top.

---
*Sources: [[raw/why-notification-systems-are-a-nightmare]], [[summary/why-notification-systems-are-a-nightmare]]*
*Last updated: 2026-11-20*
