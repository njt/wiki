# Queues Don't Fix Overload

Fred Hebert's 2014 classic on why slapping a queue in front of a slow system is almost always the wrong answer. The core metaphor is a bathroom sink: queues handle short bursts fine (the basin fills, then drains), but under sustained overload the queue just defers an inevitable and more catastrophic failure. Hebert calls the true bottleneck the "red arrow" -- a hard limit (database, API, disk, bandwidth, CPU) that no amount of local optimization can bypass. The article was written in the context of Erlang systems, but the argument is language-agnostic and has aged perfectly.

The two honest choices for overload are back-pressure (blocking input, letting callers slow down) or load-shedding (dropping work, telling callers to retry). Everything else -- bigger queues, more workers, persistent queues, fancier retry logic -- is just making failures rarer but more catastrophic. The slowness you're trying to eliminate was the canary.

---

## Key Quotes

> "Someone should have picked what had to give: do you stop people from inputting stuff in the system, or do you shed load."

This is the article's thesis in one sentence. Every overload scenario comes down to this binary choice. Most teams pick neither, defaulting to "buffer it and hope" -- which Hebert argues is the only genuinely wrong answer.

> "All of this was just premature optimization."

Hebert's diagnosis of the typical queue-adoption story: engineer sees slowness, introduces a queue, gets instant wins, queue fills up, adds workers, queue still fills, system dies. The optimization was premature because nobody had identified the true bottleneck first.

> "Nobody considered what the true, central business end of things is, and what its limits are."

The article's sharpest critique of engineering culture: we reach for infrastructure solutions before understanding the system's actual constraints. Hebert argues you should identify the bottleneck first, then ask it for permission to send more data -- not buffer around it.

> "In the end the queue just makes things worse."

Brutal and honest. Queues transform slow, graceful degradation into fast, catastrophic failure. You trade predictable overload behavior for uncertain meltdown under peak load.

> "When it goes bad, it goes really bad, because everyone tried to close their eyes shut and ignore the fact they built a dam to solve flooding problems upstream of the dam."

The dam metaphor captures the temporal dimension: queues separate cause from effect in time, which makes the connection invisible to operators. By the time the queue melts down, the overload event that caused it is ancient history.

---

## Key Themes

#concept #pattern #distributed-systems #reliability

**The Red Arrow.** Hebert's term for the true bottleneck: the component with a hard throughput limit that everything else depends on. Finding it is the first step of any overload response. Optimizing anything else is rearranging deck chairs.

**Back-Pressure vs. Load-Shedding.** The only two honest responses to overload. Back-pressure (also called flow control) propagates slowness upstream so callers self-throttle. Load-shedding (also called selective admission) drops low-priority work to protect high-priority work. Hebert's real-world analogies: "bouncers in front of a club, water spillways to go around dams, the pressure mechanism that keeps you from putting more gas in a full tank."

**Queues as Optimization, Not Architecture.** Queues are legitimate for smoothing transient bursts. They become catastrophic when elevated to an architectural primitive that hides systemic overload. Hebert's one allowed use case: inter-process messaging in languages that lack proper IPC (PHP, Ruby), and even then he calls it "marginally better than using a MySQL table." [[ZeroMQ — Universal Messaging Library]] is that legitimate use case built out to full strength — a whole library whose job is carrying atomic messages over sockets with pub-sub, push-pull, and request-reply patterns, and whose pitch stops at transport and topology rather than promising durability, which is what keeps it a queue instead of a database masquerading as one.

**The End-to-End Principle.** Persistent queues break the end-to-end principle by creating fire-and-forget boundaries where errors can't propagate back to callers. Hebert ties this to idempotency: an idempotent API lets callers safely retry, which means you can shed load with confidence that dropped work will be retried successfully.

---

## Critical Analysis

This article has aged absurdly well. Ten years later, every cloud provider sells queue services (SQS, Pub/Sub, EventBridge), every microservice architecture reaches for queues as the default integration pattern, and every incident postmortem includes "we'll add a queue" as a remediation action. Hebert's 2014 argument is more relevant now than when he wrote it.

The article's strength is its clarity about what queues actually do: they decouple producers from consumers in time. That decoupling is sometimes exactly what you want (smoothing bursts, handling variable processing times). But it also decouples cause from effect, which makes overload invisible until the queue is full and everything falls over at once. The operational nightmare isn't the failure -- it's that nobody can trace the failure back to its root cause because the queue swallowed the evidence.

Where the article falls slightly short: it doesn't fully engage with the legitimate operational reality that back-pressure isn't always feasible. If you're consuming from a third-party API that rate-limits you, you can't propagate back-pressure to it -- a queue is the only option. Hebert acknowledges this implicitly (the "red arrow" can be external), but the prescription of "ask the bottleneck for permission" assumes the bottleneck is under your control.

The Erlang/BEAM context matters more than is obvious on first read. BEAM processes are so cheap that you can have a process per request and let the VM's preemptive scheduler handle concurrency -- natural back-pressure emerges from the runtime. In systems without that property (most of them), the queue-as-bandaid trap is even more seductive because the alternative requires building back-pressure infrastructure that BEAM gives you for free.

The article pairs well with Tricot's [[Event-Driven vs Polling Architectures]] -- both argue that the infrastructure choice (queue, webhook, poll) should follow from the system's actual constraints rather than being the default answer. It also resonates with [[Designing a Passively Safe API]]'s insistence on idempotency as the foundation for safe retry, and with the [[Distributed Systems]] hub's observation that agent orchestration keeps reinventing distributed systems primitives without learning their lessons.

El-Deeb's [[Hidden Inefficiencies Behind Delivery Delays]] extends Hebert's argument from infrastructure queues to organizational ones: review queues, approval queues, and coordination queues are the same phenomenon -- visible wait times that are downstream symptoms of invisible bottlenecks (reviewer scarcity, unclear ownership, weak specs), not problems to be solved with more queue management.

[[State-Oriented Consistency]] diagnoses the same structural error in a different domain: reaching for strong consistency everywhere (like reaching for queues) without first identifying what each piece of state actually requires. The Keel IoT team's "Uniform Consistency" is a close cousin to Hebert's queue-as-default — both are trusted mechanisms applied by habit rather than by need.

[[The Wicked Reason Removing Code Beats Better Scheduling]] extends the same structural error to frontend performance: resource scheduling/reordering is the queue — it defers bytes rather than eliminating them, and in the tail of the connection curve the deferred work still lands on the main thread as "thuds" that show up in INP data. Removing code is the load-shedding equivalent — cut the bytes on the wire rather than resequence them.

Autoscaling is the capacity-side cousin of this argument: instead of back-pressure or load-shedding, you add pods to absorb the load — but [[The Component Substitution Fallacy]] shows the red arrow can hide in a place your scaling policy never watches. In GitHub's 2026 outage a saturated Istio sidecar went unscaled because the policy only watched host load, and Hochstein's deeper point is that fixing that component still wouldn't explain the outage: the failure was an interaction of traffic, policy, sidecar saturation, retry logic, and HAProxy load, not one broken part.

---

*Source: Fred Hebert, [ferd.ca](https://ferd.ca/queues-don-t-fix-overload.html), November 19, 2014. Fetched 2026-06-12.*
