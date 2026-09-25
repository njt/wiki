---
url: https://blog.verygoodsoftwarenotvirus.dev/posts/2026/09/23/why-didnt-anybody-tell-me-about-hash-slots
date_fetched: 2026-09-25
---

# Why didn't anybody tell me about hash slots

I work on a service that decides which courier gets offered your delivery at a gig economy delivery app. When you open the app and ask for something to be brought to you, some code somewhere has to decide which of the couriers should be offered the job. We’re the ones who make those offers.

I’m not at liberty to discuss the myriad parameters that go into the decision, but a pretty obvious one you can suss out externally without knowing much is distance. To make good matches, we need to know *how far away is everybody, really*. Straight-line distance will tell you it takes ten minutes to get across a river with no bridge. For that we lean on a routing engine, which is far and away the biggest contributor to our own latency. We have very specific latency targets that need to be respected. We’re basically always trying to empty a bucket in a set amount of time, and when the bucket overflows, everybody gets wet.

As a heads up, I’m not at liberty to discuss specific figures or terms we use at work. I can say that a service sped up tremendously, I can’t say it went from 600ms to 100ms, for example.

# The shape of the problem

A busy area needs a tremendous amount of route estimates to get through a single pass at assigning work, and it needs them again continuously, for as long as the business is running. Handling this scale is a constant concern.

The useful thing about those estimates is that they’re frequently repetitive. It doesn’t take the guy three houses down from me any meaningful amount more or less time to get to the grocery store than it takes me, and neither of us could tell the difference between driving to that grocery store and driving to the gas station in its parking lot. Restaurants, meanwhile, do not move at all. Two couriers a block apart produce two nearly identical route requests, and thirty seconds later, from two new positions, they produce two more.

Without any caching layer between us and the routing engine, you’re invariably duplicating work, re-calculating what are functionally the same distances on a regular cadence for minutes at a time.

A process-local cache also doesn’t work here. The process that computes an estimate is rarely the one that needs it next, so the cache has to be shared.

# The naive approach

Caching on raw coordinates is a non-starter. You need to use H3 to dedupe the coordinates. H3 breaks the world up into differently-sized hexagons, and allows you to fetch the hex’s ID from a given coordinate pair.

The resolution is a lever. Too wide a hex size leads to higher hit rates, and less accurate estimates. Too small a hex effectively swaps one coordinate identifier for another.

So the key was `<origin hex>:<dest hex>:<resolution>`, the value was the estimate, and the write path was an `MSET`. Easy peasy, right?

# Not so fast, there

We use a Redis *cluster*, and clusters have to evenly spread keys and their values.

Redis creates 16,384 hash slots, and distributes them among the cluster’s nodes. Commands dealing with multiple keys like `MSET` or `MGET` are only valid if every key lands into the same slot.

I found this out by looking at traces. A single read was showing up as scores of **individual  MGET spans for a single key each**. And that’s considering that the Otel collector was almost assuredly dropping loads of relevant spans. Our max read latency metric was way higher than our anticipated worst case. This is how I learned slots exist.

# Slot it to me

Which slot a key lands in is determined by Redis doing `CRC16(key) mod 16384`. Two keys that differ by one character land in slots that have nothing to do with each other, which is catastrophic for batching. My keys had a unique pair of hex IDs in every one of them, so they would effectively never end up in the same slot.

A multi-key command can only address one slot, so a well-built client takes your single `MGET` of a ton of keys, groups them by the slot each one belongs to, and puts every group on its own wire. Given what I’d handed it, that was the correct thing to do. The fix is in handing it keys that don’t live in so many different slots. The write path had the same problems.

# #hashtags

Redis has an escape hatch exactly for this, called a hash tag. If a key contains a chunk of text between curly braces, then Redis hashes only the text between the braces and ignores the rest of the key. That gives us a lever to force keys into specific slots.

What matters is what you put in the braces. I’ve never owned any Bitcoin, but this problem made me think of how the Bitcoin blockchain puts meaningless numbers into its blocks so the end hash matches some property. We basically needed that here, but with much simpler requirements for the output hash.

I opted for an integer, found like blockchain miners by brute force. The tag is really a template, something like `{routing:v1:<n>}`, and at startup we iterate up from zero and check the hash on the whole tag each time to see which slot it lands in. We do this until every node has as many as we need.

Once the tag is doing the work, a batch of thousands of keys can be split into a handful of legal multi-key commands aimed at exactly one node.

# Sorting keys before you fan out

Each Redis node executes commands on a single thread, so 20 concurrent requests aimed at the same node are functionally a queue. Collapsing the fan-out reduced the quantity of `MGET` requests we had to make, but didn’t preclude us from spamming a given node with them.

If the chunks aren’t organized by destination, several of them will target the same node at the same time, and that node will do them one after another while other nodes sit idle. So before anything goes out on the wire, we compute the slot for every key locally and group by it. Then the fan-out is over nodes rather than arbitrary chunks, and every request is doing useful work.

# MSET doesn’t do expiry

Cached route estimates have to have an expiry, and `MSET` has no expiry argument.

The usual workaround is to pipeline a `SET ... EX` per key, which turns one command into thousands. Or you can `MSET` and then `EXPIRE` in a second pass, which doubles your commands *and* leaves a window where a crash between the two passes strands keys in the cache forever.

The answer ended up being to ask Redis to execute a script instead with `EVAL`.

# JSON is not free

The last one is embarrassing in hindsight. The cached value started as JSON, because that’s what I’ve always done. It’s a small payload, so I didn’t think it’d be that consequential a choice. At our scale, it was hogging up precious CPU time. Switching the value to a comma-separated value helped tremendously.

# So what did we learn, here?

Everything I learned in this endeavor required sussing out why the previous step failed in a way I hadn’t predicted.

Being a Go fanatic, I’m naturally reverent of Rob Pike, and in particular, his 5 rules of programming. I’ve been thinking about these rules a lot lately:


Rule 2.Measure. Don’t tune for speed until you’ve measured, and even then don’t unless one part of the code overwhelms the rest.

Rule 3.Fancy algorithms are slow whennis small, andnis usually small. Fancy algorithms have big constants. Until you know thatnis frequently going to be big, don’t get fancy.

I had measured that our interactions with the routing service were our biggest bottleneck, and thought the simple-algorithm approach for interfacing with Redis would be my huckleberry. I neglected to realize that n for us is reliably large, and I needed a fancier algorithm than I had anticipated to get the job done.

When I first encountered the issues with throughput described above, I decided to take a step back and write some benchmarking harnesses. Toy apps I could turn levers on and off in and measure the consequential throughput. Those harnesses gave me the ability to validate each approach before putting it in production, and give honest answers about what kind of performance we were buying with this increasingly complicated code.

In hindsight, I should have built the benchmarking harness first. I cut my teeth in this industry mostly at nascent startups where the default is ship now and fix later. A benchmark meant a harness, and a harness meant inventing input that resembled production, and all of *that* meant explaining to an audience that didn’t care that a 2-point ticket was now a 5.

I think in the post-Claude era of programming, where such a benchmarking harness is a matter of providing a spec and waiting 10 minutes, this is no longer acceptable. I’ve started newer endeavors by first building extensive benchmark harnesses, and it’s paid off greatly for me, so I’m trying to make it a habit.
