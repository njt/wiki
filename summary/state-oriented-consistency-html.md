---
url: https://keel-iot.eu/blog/state-oriented-consistency.html
title: State-Oriented Consistency
author: Keel IoT team
date_published: 2026
date_fetched: 2026-08-07
topics:
  - misc
---

A team building a clustered MQTT broker (Keel MQTT Gateway) discovers the hard way that asking "which consistency model should the cluster use?" is the wrong question. The right one: "which consistency guarantee does *this specific piece of state* actually need?"

The discovery came via an OOM kill incident where every pod boot loaded *every client's* stored session state — the entire fleet — because the architecture assumed every node needed to be ready to serve any client. The root cause wasn't a memory leak; it was an unexamined architectural assumption the authors name **Uniform Consistency**: the habit of applying one consistency strategy to the whole system without asking each piece of state what it actually requires.

The article presents a five-row classification table mapping different kinds of cluster state (live sessions, offline sessions, message routing, durable messages, membership) to their minimum sufficient guarantees, then to concrete mechanisms (coordinator, deterministic placement, AP routing, durable shared storage, gossip). The insight isn't any single row — it's that a system built by capable people will still default to Uniform Consistency unless something forces the question onto the table, row by row.

The generalizable framework is a five-step loop: identify the state → ask what happens if two nodes disagree → find the minimum guarantee → choose the weakest mechanism that provides it → repeat. The article closes by noting consensus was recognized as a solved problem; the hard decision was identifying *which parts* deserved it, and refusing to let it creep into rows that didn't need it.
