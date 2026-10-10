# I Tested 11 HTTP Resilience Libraries

Gábor Koos runs a 21-scenario conformance suite against eleven JavaScript fetch wrappers offering resilience patterns (retry, timeout, circuit breaker, bulkhead, hedge, dedupe), finding that every library nails single-policy basics while combinations — abort during backoff, cancellation in a queue, a deadline spanning retries — produce most of the 25 failures. The suite exists because fixing correctness bugs in his own ffetch turned an elegant small library into a huge one, and he wanted to know whether anyone had done better.

---

## The shape of the result

The matrix is blunt: 55 of 147 implemented cells are the five trivial retry scenarios every library passes. The failures cluster with almost poetic uniformity around one pattern, which the author names directly:

> The 25 failures cluster around a single pattern: something else arrives while a policy is waiting: a cancellation during a backoff sleep, a deadline that expires during the wait, a queued caller that leaves, a probe that completes after the breaker reopened.

That sentence is the whole thesis of the piece. Resilience policies are specified and tested in isolation, but they are *deployed* concurrently, and the interleavings are where correctness lives. This is textbook distributed-systems epistemology applied to a client-side library — the same insight that drives Jepsen.

## Cancellation as a contract

The starkest finding is fetch-retry's retry delay: a bare `setTimeout(function () { wrappedFetch(++attempt); }, delay)` that never reads `init.signal`. The caller aborts, waits out the backoff anyway, and watches two more attempts leave. Koos is unsentimental about it:

> A caller aborting during the wait cannot interrupt the sleep, so it waits it out and then watches another attempt leave.

Similarly, four of five bulkhead queues hold capacity for callers who already left — "losing a slot to a caller that already left is a leak rather than a design decision." His methodology makes an explicit point of being stricter than documentation:

> These requirements can be stricter than a library's documentation, so a failure records a disagreement with the scenario's rule even when the library makes no promise about that interaction.

This is a defensible and arguably the only useful stance for a conformance suite — an undocumented non-guarantee is still a bug from the caller's perspective — though it does blur the line between "incorrect" and "undocumented."

## The author grades his own homework

The most admirable section is the one most bloggers would have cut:

> Because a lot of these scenarios come from bugs I had already fixed in ffetch, my library enters the comparison with an advantage. Its 20 passes out of 21 scenarios partly reflect that history and the behaviour I chose to implement.

He still publishes the one ffetch failure in detail: the circuit plugin clears its `isOpen` flag on any late success without checking which cooldown it belongs to, because there is no generation counter — "the stale-success rule is the mirror image" of a recheck the plugin *does* implement. It's a rare public admission that also doubles as a design lesson: the bug and its fix are the same abstraction at two different layers.

## Opinionated take

This is what library comparison should look like and almost never does: pinned versions, published scenario configurations, assertions linked per cell, a weekly rebuild that refuses to publish incomplete runs, and an explicit confirmation-bias disclosure. The usual library benchmark is a throughput chart; this one asks "does your promise settle when the caller aborts," which is the question that actually pages someone at 3am.

Two caveats. First, the confirmation-bias problem is real beyond the disclaimer: the entire scenario vocabulary came from one library's bug history, so "combinations" means *the combinations ffetch needed*. A suite built from hedging bugs in someone else's library might rank the field differently. Second, the piece is also an origin story for a design argument — correctness won over simplicity, and the author's conclusion is not "complexity is fine" but "complexity is the honest price, and I'm still not confident I paid it correctly":

> Working through these cases has made me more willing to accept complexity in ffetch, but also more careful about assuming that the complexity means I have got everything right.

That last clause is the correct amount of humility. The suite proves not that ffetch is right but that eleven other libraries are measurably wrong in enumerable ways — which is still the most useful thing a benchmark can say.

## Related pages

This source strengthens [[Computers Are Bad, Actually (Kyle Kingsbury, Jepsen)]] — Jepsen's project of testing what systems *do* under interleavings rather than what they claim, here brought down from distributed databases to a client-side fetch wrapper. It nuances [[21 Years and Counting of Eight Fallacies of Distributed Computing]]: the fallacies presume a reliable network, and these libraries show that even the code explicitly built to compensate for unreliability mishandles its own composition. And it complicates [[How SQLite Tests Software]]: both are arguments for scenario-driven correctness testing over feature checklists, but Koos's suite is explicitly subjective and author-biased where SQLite's is a decades-old invariant machine.

---

*Sources: [[raw/2026-10-04-i-tested-11-http-resilience-libraries]], [[summary/2026-10-04-i-tested-11-http-resilience-libraries]]*
*Last updated: 2026-10-10*
