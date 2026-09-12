# The Valley of Webhooks

A deeply informed polemic against using webhooks for data replication between services, framed through the author's experience building the same system three times. The core argument: webhooks are notifications ("something happened, here's a POST"), and using them to maintain a correct local copy of a provider's data is a category error. The industry has spent fifteen years building compensating infrastructure on the valley floor — dedup tables, signature verification, retry queues, reconciliation crons, local tunnel CLIs — rather than questioning whether the primitive itself is wrong. The proposed alternative, SCROLL, inverts the arrow: instead of the provider pushing events at you, you pull a cursor-addressed change log at your own pace, eliminating the endpoint, the signatures, the dedup table, and the bootstrap importer in one move.

---

## Key Quotes

> "I want to be honest about what that cron is. It's a written confession. It says: *I do not trust the copy I built, and I have no way to know when it's wrong, so I will re-derive it from scratch every night, forever.*"

This is the emotional core of the piece and its best line. The reconciliation cron isn't a backup — it's an admission that the system lacks any internal mechanism for verifying correctness. Trust has decayed into a scheduled full rebuild, and the author names this without euphemism.

> "It's a jigsaw puzzle where the manufacturer had the original picture, cut it up, mailed me the pieces one at a time, lost a few in the post, mailed some twice, and printed nothing on the box."

The jigsaw metaphor captures the absurdity: the provider *has* the ordered, complete log — it's what renders their own dashboards — but the protocol forces every consumer to reassemble it independently from lossy, unordered POSTs. Every consumer writes the same bugs.

> "None of this is any provider's bug. Their webhooks work exactly as documented. The problem is what a webhook *is*: a notification, 'something happened, here's a POST about it.' Notifications are a fine way to trigger a side effect and a terrible way to transfer a dataset, and somewhere along the way we started using them for the second thing without noticing we'd changed jobs."

The distinction between "trigger a side effect" and "transfer a dataset" is the analytical contribution. It reframes the entire webhook ecosystem not as buggy implementations but as a tool being used for a job it was never designed for. The author is careful not to blame providers — their webhooks do exactly what they promise — while still indicting the industry for conflating the two jobs.

> "The pile has an economy on top of it. Svix exists so providers don't have to build webhook delivery; Hookdeck exists so consumers don't have to build webhook ingestion. AWS will sell you the valley as managed services... And an entire industry of connector platforms (Fivetran, Airbyte, every 'unified API' startup) is, at bottom, pseudo-CDC: change data capture reconstructed from webhooks and polled list APIs, one bespoke connector at a time, sold as a product."

The local optimum diagnosis in economic form. These are good products doing excellent engineering — and their existence is evidence that the underlying primitive is broken. Real CDC inside a database is a solved problem because there's a log. Between companies, we rebuild it from doorbells.

> "My favorite workaround of them all is the local tunnel. Many providers ship a CLI like `stripe listen` that opens a tunnel to your laptop, because a webhook cannot reach localhost. Think about what that is: a product, built and maintained by the provider, reinvented multiple times, whose entire purpose is to work around the delivery direction of their own primitive."

The local tunnel as reductio ad absurdum. When multiple providers independently build the same tool whose sole purpose is to route around the architectural limitation of their own API primitive, the primitive is answering the wrong question.

> "Inside a database, capturing changes is a solved problem: it's called replication, and it works because there's a log. Between companies, we rebuild it out of doorbells."

The one-sentence summary of the whole argument.

## Key Themes

- **#concept Local optimum** — Borrowed from evolutionary biology's fitness landscape: webhooks-for-replication sit on a small hill that's better than its immediate alternatives, so the industry climbs there and stays, even though a much higher peak (consumer-pull change logs) exists across the valley. Getting there requires crossing temporarily worse designs, and evolution doesn't do temporarily worse.
- **#pattern The webhook stack** — The set of compensating infrastructure that accretes around any webhook-based replication system: signature verification, dedup tables, ordering buffers, bootstrap importers with locking schemes, and a reconciliation cron. The author argues this stack is not accidental complexity but evidence of a category error.
- **#concept Notifications vs. data transfer** — The two jobs webhooks are asked to do. Job 1 (trigger a side effect: send receipt, start build) is what webhooks were born for. Job 2 (keep a copy of provider data correct) is the one where every property webhooks lack — ordering, completeness, bootstrap, verifiability — is precisely the property you need.
- **#concept Convergent evolution toward the log** — Stripe's `/v1/events` (ordered, listable), WorkOS's Events API (cursor-paginated, recommended over webhooks for consistency) — providers keep half-building the log themselves, each with bespoke cursor semantics and no shared contract. The direction is unmistakable even if the standard doesn't exist.
- **#pattern SCROLL (Synchronized Change Replication Over Line Logs)** — The author's draft protocol: a single `GET /feed/<collection>?cursor=X` endpoint returning NDJSON with upsert/delete operations, supporting both streaming (`Prefer: stream`) and polling modes from the same consumer code. Eliminates the endpoint, signatures, dedup table, ordering buffer, and bootstrap importer. Carries an optional checksum at log-end for verifiability.
- **#tool Webhook ecosystem** — Svix (provider-side delivery), Hookdeck (consumer-side ingestion), EventBridge + SQS + Lambda (AWS's managed valley), Fivetran and Airbyte (pseudo-CDC sold as product), `stripe listen` and equivalent local tunnels (workarounds for the push direction).
- **#pattern Fitness landscape thinking** — The valley metaphor applied to API design: populations climb whatever slope they're standing on, parking on local optima because crossing the valley means temporarily worse designs. The author uses this to explain why the industry has spent fifteen years compensating rather than redesigning.

## Critical Analysis

**The core diagnosis is sharp and specific.** Distinguishing "trigger a side effect" from "transfer a dataset" is an analytical knife that cuts cleanly through the webhook discourse. It explains why webhooks are perfect for CI triggers and terrible for user provisioning without blaming any provider. It also explains the shape of the compensating infrastructure: every piece of the webhook stack (dedup, ordering, bootstrap, reconciliation) corresponds to a property that the trigger-a-side-effect primitive lacks and the transfer-a-dataset job requires.

**The local optimum framing is powerful but underspecified.** The author borrows the concept from evolutionary biology without fully mapping it to software economics. In biology, populations can't cross valleys because intermediate forms are less fit and selection eliminates them. In software, we can and do cross valleys — we migrated from SOAP to REST, from monoliths to microservices, from VMs to containers. The question isn't whether the valley can be crossed but what the forcing function would be. The author gestures at this with Stripe and WorkOS as convergent evolution, but doesn't name what would make SCROLL adoption rational for the next provider.

**The SCROLL proposal is elegant and essentially correct, but the hard problems are organizational.** The author acknowledges that "nobody serves this today" and that a shim can synthesize a feed from existing webhooks and list APIs. But the real obstacle isn't technical — it's that webhooks are the cheapest thing a provider can ship, and a consumer-pull change log requires the provider to maintain ordered history, manage cursor state, and commit to retention windows. That's infrastructure cost shifted from consumer to provider, and the consumer has no leverage to demand it. The protocol draft is necessary but not sufficient; the missing piece is the economic argument that would make providers want to ship it.

**The fitness landscape metaphor cuts both ways.** The author uses it to argue we're stuck on a local optimum. But one person's local optimum is another's efficient market equilibrium: if the compensating infrastructure (Svix, Hookdeck, Fivetran) delivers acceptable reliability at acceptable cost, the valley IS the peak — the industry found a working division of labor and built tooling around it. The argument that we *should* climb higher needs to show that the current equilibrium is unstable or that the higher peak offers enough additional value to justify the transition cost. The author makes the technical case beautifully but doesn't fully make the economic one.

**The article's greatest strength is its concreteness.** The author traces a specific afternoon-that-grew into a week, names every component in the stack, walks through what SCROLL eliminates and why. The jigsaw metaphor, the local tunnel as reductio ad absurdum, the reconciliation cron as written confession — these are memorable because they're specific. Compare with [[Event-Driven vs Polling Architectures]], which makes a similar argument (webhooks need reconciliation) but stays at the architectural level. This piece goes deeper: it argues webhooks are the wrong *category* of tool, not just insufficient alone.

**The unspoken thread is organizational memory.** The author built the same system three times because each company independently discovered it needed webhook-based replication, and each time the engineer on the ground had to rediscover the full stack. The protocol proposal is also a proposal about knowledge transfer: a standard change-log format means the fourth person doesn't have to learn the same lessons the hard way. This connects to [[The New Software Lifecycle]] and the idea that context engineering is the financial lever — the webhook stack is context that every team pays to reconstruct.

**What this means for agents.** The article was written before the agentic coding explosion, but it's directly relevant. Agents integrate with external services through their APIs, and those APIs overwhelmingly offer webhooks for change notification. An agent that subscribes to Stripe webhooks to track customer state inherits the entire valley — dedup, ordering, reconciliation — in its tool definitions and state management. The SCROLL proposal, if adopted, would make agent-to-service integration dramatically simpler: a single GET loop replaces a route, a signing secret, and a reconciliation cron. This is the same argument made in [[AI-Ready APIs — Postman AWS Competency]] from the other direction: API quality, not model capability, is the bottleneck.

---

## See Also

- [[Event-Driven vs Polling Architectures]] — Tricot's architectural companion: webhooks need reconciliation, and the source picks the pattern
- [[Postgres CDC in ClickHouse, A Year in Review]] — CDC inside a database is solved; between companies, we're still rebuilding it from doorbells
- [[Signals — The Push-Pull Algorithm]] — The push-to-pull inversion at the UI framework level; the same architectural move at a different scale
- [[Your Backend Is Full of Hidden Workflows]] — The webhook stack is the hidden workflow that accretes around every SaaS integration
- [[AI-Ready APIs — Postman AWS Competency]] — API quality as the bottleneck for agentic AI; webhook semantics are part of that quality
- [[Queues Don't Fix Overload]] — Queues treat symptoms not causes; the webhook stack treats the symptoms of a wrong primitive
- [[Databases and Data]] — Hub page for data replication, CDC, and storage patterns

---

*Sources: [[raw/the-valley-of-webhooks]], [[summary/the-valley-of-webhooks]]*
*Last updated: 2026-08-06*
