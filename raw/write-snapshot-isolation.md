---
url: https://remy.wang/blog/si.html
date_fetched: 2026-07-05
backfilled: true
---

Snapshot isolation (SI) is a popular approach to concurrency control
in databases systems. It avoids many forms of anomalies, yet provides a
high degree of concurrency, especially for read-heavy workloads. Yet, as
shown by a classic paper (Berenson et al. 1995), SI does
*not* guarantee *serializability*. A concurrent execution
of a set of transactions is said to be serializable if it is equivalent
to executing the transactions in sequence. Serializability is therefore
the strongest guarantee of correctness for database transactions,
analogous to sequential consistency in general concurrent programming.1 This has led to attempts to "patch
up" SI, including *serializable snapshot isolation* (SSI) (Cahill, Röhm, and Fekete
2009) which adds addtional checks on top of the standard SI
implementation. SSI has been adopted by mainstream systems including
PostgreSQL. Personally, I find SSI to be unsatisfying, as it feels like
a bolt-on solution to fix the already-broken SI. I was therefore
delighted to stumble upon *A Critique of Snapshot Isolation*
(Yabandeh and Gómez
Ferro 2012) which describes a method to fix SI by addressing
the root cause and guarantee serializability, all by changing one single
line of code. I'll distill the core idea of WSI in the rest of this
post, and hopefully after reading, you'll feel like you could have come
up with the algorithm yourself.

Another benefit of SI is that it's so simple to implement: you fork the DB at the beginning of the transaction, run your queries over your fork, then try to merge the DB back in. If you've used git, this will feel just like merging a branch, and you'd be right to feel that. Under SI, each DB item (say a row) has a timestamp recording the last committed update, then for each transaction at commit time, we check if each item the transaction wants to write has been overwritten since the transaction started. If so, the transaction aborts. In other words, SI avoids writing over "stale values".

For example, consider the following sequence diagram, where
T1 sets Z to X+1, and T2 sets Z to Y+1. Because
T2 updated Z as T1 is running, when T1
wants to commit, the Z value it sees has already become stale, so
T1 aborts. Concretely, T2 would increment Z's
timestamp when it commits, and T1 will see that the last
updated time of Z is after T1's own start time.

For another example, suppose T1 wants to set Y to X+1, and
T2 wants to set X to Y+1. As the diagram below shows, neither
of the transactions is writing over stale value, so both of them
succeed.

The pair of examples also demonstrate the issue with SI: the first
execution is serializable, yet aborted by SI, whereas the second is
allowed by SI, but not serializable! In particular, the first example is
equivalent to the serial execution where T2 runs before
T1, in which case T1 would overwrite T2
anyways. In the second example, both X and Y would be incremented by 1
at the end of execution, but in any serial schedule, one of them would
be incremented by 1, and the other one by 2. Overall, this means SI both
forbids certain correct executions (false negatives) and permits
incorrect ones (false postives).

The issue of snapshot isolation stems from the conflict check: SI
checks for *write-write* conflicts, and aborts upon finding one.
But that doesn't really make sense! In fact, you may have felt slightly
uneasy when reading "stale writes" above - if a value becomes stale, why
is it bad to overwrite it? Because it is not! What we really should be
worrying about is *stale reads*. Intuitively, a transaction would
only read a value to calculate what values it should write. Therefore, a
stale read would mean the written values based on that read were
calculated using outdated data, thereby resulting in inconsistencies in
the database. The write-snapshot isolation (WSI) algorithm following
(Yabandeh and Gómez
Ferro 2012) checks exactly for such stale reads. During WSI
execution, we still record the last-updated time for each DB item. But
for each transaction T that is about to commit, we check if any value
read by T has been overwritten, i.e., become stale. If so, T aborts.

Let's revisit the examples. In the first example, because X has not
been updated since T1 read it, the value is still fresh, and
T1 successfully commits, even though T2 has
written to X in the mean time. T2's write is simply
overwritten, and the execution is equivalent to a serial one where
T1 runs after T2.

In the second example, because T2 updated X while
T1 is running, the value of X read by T1 has
become stale when T1 is ready to commit, so T1
must abort.

In general, Yabandeh and Gómez Ferro (2012) proves
that WSI guarantees serializability, but also forbids certain
serializable executions. In other words, WSI still under-approximates
serializability (w.r.t. the set of allowed execution schedules). To show
this, the authors gave an example schedule that is serializable yet
forbidden by WSI. But, upon a closer look, the serial schedule
equivalent to the given example reverses the commit ordering! In other
words, the given example violates *strict serializability* which
requires the commit order to be preserved. So one might ask: is WSI
necessary for strict serializability? I believe the answer is yes,
assuming a lock-free implementation based on MVCC. (Dis)proving this
formally is left as an excercise to the reader :)

Yeah, I wonder that too. If WSI is so simple and elegant, why doesn't everyone implement it? One factor is timing: when (Yabandeh and Gómez Ferro 2012) was published, PostgreSQL already patched their SI implementation with additional checks following SSI, thereby guaranteeing serializability. And since that was a bolt-on solution, they did not have to change the SI implementation itself. In contrast, although WSI is a single-line diff on paper, implementing it in a real system can be more involved. On the other hand, SI itself works well enough despite not guaranteeing serializability, whereas WSI will necessarily abort some transactions that would be allowed under SI. This is hinted at by a later paper (Gómez Ferro et al. 2014) by Ferro and Yabandeh that surprisingly does not mention the WSI work at all. In any case, I think people should consider WSI if they are building a new DB system from scratch. I will certainly keep teaching it because it's so beautiful!

Strict serializability additionally requires that the transaction order is preserved, analogous to linearizability.↩︎
