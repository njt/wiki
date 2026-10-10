---
url: https://blog.gaborkoos.com/posts/2026-10-04-I-Tested-11-Http-Resilience-Libraries/
title: "I Tested 11 HTTP Resilience Libraries"
author: Gábor Koos
date_fetched: 2026-10-10
date_published: 2026-10-04
topics:
  - distributed-systems
  - software-engineering-craft
---

Gábor Koos built a 21-scenario conformance suite for HTTP resilience patterns — retries, timeouts, circuit breaking, bulkheads, hedging, deduplication — and ran it against eleven fetch-wrapper libraries including his own ffetch. The suite grew out of bugs he found in ffetch, when correctness fixes turned his deliberately minimal library into something much bigger.

The headline result: every library handles basic retry cases (recovery, exhaustion, body replay, retries over real HTTP), but combinations are where things fall apart. 25 of 84 implemented failure cells share one pattern — something else arrives while a policy is waiting: an abort during a backoff sleep, a deadline expiring mid-wait, a queued caller cancelling, a stale success arriving after a breaker reopened. Six libraries keep retrying after the caller aborts; four of five bulkhead queues leak capacity to cancelled waiters; several dispatch attempts after the caller's deadline has passed.

Koos is candid about confirmation bias: the scenarios come from bugs he already fixed, so ffetch's 20/21 score partly reflects his own choices. Every cell links to its configuration and expected behaviour, the matrix is rebuilt weekly from pinned versions, and he explicitly invites challenges to his interpretations. Hedging turns out to be the least-finished surface — 32 of 84 N/A cells — and even his own library fails the stale-success circuit scenario for lack of a generation counter.
