---
url: https://moq.dev/blog/webrtc-is-the-problem/
title: "WebRTC Is the Problem"
author: "@kixelated"
date_fetched: 2026-05-18
date_published: 2026-05-06
---

# OpenAI's WebRTC Problem

By @kixelated (moq.dev), published 2026-05-06.

## Summary

The author, a self-described "Certified WebRTC Expert" who built WebRTC SFUs at Twitch and Discord, argues that WebRTC is fundamentally wrong for Voice AI. The core thesis: WebRTC's aggressive packet dropping for real-time latency, its ~8 RTT connection setup, its port-binding architecture that breaks under NAT/roaming, and its browser implementation being "owned by Google and tailor made for Google Meet" make it a poor foundation. The alternative: QUIC/WebTransport, which offers single-RTT connection establishment, connection migration via CONNECTION_ID, stateless load balancing, and head-of-line blocking that the author argues is actually desirable for voice AI.

## Full Content

### Opening Hook

The author says the OpenAI blog post "triggered me more than it should have" and that his "meaty fingers" wanted to respond. Core thesis: "You should NOT copy OpenAI" and "I don't think you should use WebRTC for voice AI. WebRTC is the problem."

### Author Background

The author built a WebRTC SFU at Twitch about 6 years ago, originally using Pion (Go) like OpenAI, but forked it after benchmarks "revealed that it was too slow." He ended up rewriting "every protocol." A year ago at Discord, he rewrote the WebRTC SFU in Rust.

WebRTC "consists of ~45 RFCs dating back to the early 2000s" plus de-facto standards still in draft. He calls himself a "Certified WebRTC Expert" — which is why he "never, never want[s] to use WebRTC again."

### Product Fit

"WebRTC is a poor fit for Voice AI." The author acknowledges this seems counterintuitive since WebRTC is for conferencing and involves speech.

### WebRTC Is Too Aggressive

The author imagines speaking a prompt about walking vs. driving to a car wash. "WebRTC is designed to degrade and drop my prompt during poor network conditions." His argument: users would "rather wait an extra 200ms" for their prompt to arrive intact, since a garbage prompt yields a garbage response and LLMs are "not particularly responsive anyway."

"It's impossible to even retransmit a WebRTC audio packet within a browser" — they tried at Discord. The implementation is "hard-coded for real-time latency or else."

UPDATE: Some WebRTC folks say this may be a skill issue; audio NACKs might be possible with the right SDP munging, but "the WebRTC jitter buffer is aggressively small."

### TTS Is Faster Than Real-Time

Pipeline: microphone → server → GPU generates TTS. Example: 2s of GPU time to generate 8s of audio. Ideally, the client would buffer streamed audio so network blips go unnoticed.

But "WebRTC has no buffering and renders based on arrival time" — timestamps are "just suggestions." To compensate, OpenAI must "add a sleep in front of every audio packet before sending it." If congestion hits, the packet is lost permanently.

Analogy: "It's the equivalent of screen sharing a YouTube video instead of buffering it. The quality will be degraded."

Fun fact: WebRTC's dynamic jitter buffer adds 20ms–200ms of latency, none of which is needed if you transfer faster than real-time.

### Ports Ports Ports

TCP connection identification via source/destination IP and ports. Problem: client addresses change (WiFi → cellular, NATs remap). When that happens, "bye bye connection" — requiring an expensive TCP+TLS handshake of 2–3 RTTs.

WebRTC tried to solve this: an implementation "is supposed to allocate an ephemeral port for each connection" so sessions are identified by destination IP/port only. If source changes, "that's still Bob because the destination port is the same."

But at scale: "Servers only have a limited number of ports available," "Firewalls love to block ephemeral ports," and "Kubernetes lul."

### Hacks by Necessity

Most services ignore WebRTC specs and mux multiple connections onto a single port. At Twitch, the author hosted WebRTC on UDP:443 (normally the HTTPS/QUIC port) — lying about the port helped bypass firewalls. Discord uses ports 50000–50032 (one per CPU core), which gets blocked on more corporate networks.

The routing problem:
- STUN: route on unique ufrag
- SRTP/SRTCP: browser chooses random ssrc (u32) — usually routable
- DTLS: "Uh oh" — hopes for RFC9146 support
- TURN: hasn't implemented it

OpenAI uses only STUN: "Relay parses only STUN headers/ufrag; it uses cached state for subsequent DTLS, RTP, and RTCP." The author calls this a positive spin on: "We really hope the user's source IP/port never changes, because we broke that functionality."

Fun fact: Browsers can generate duplicate ssrc values. When a collision occurs with no source IP/port mapping, Discord tries decrypting packets with each possible key to identify the connection.

### Round Trips

The OpenAI blog listed "Fast connection setup" as a requirement. The author responds: "lol."

Minimum 8 RTTs to establish a WebRTC connection:

Signaling server (WHIP):
- 1 for TCP
- 1 for TLS 1.3
- 1 for HTTP

Media server:
- 1 for ICE (with server)
- 2 for DTLS 1.2
- 2 for SCTP

He notes it's "complicated to compute" since some protocols can be pipelined. He calls it "nonsense" because WebRTC needs to support P2P — even with a static-IP server, "you still need to do this dance." When signaling and media run on the same host, you do "two redundant and expensive handshakes."

### Forking the Protocol

"WebRTC practically encourages you to fork the protocol." The browser implementation is "owned by Google and tailor made for Google Meet" — an "existential threat for conferencing apps."

"That's why every conferencing app (except Google Meet) tries to shove a native app down your throat. It's the only way to avoid using WebRTC."

The author thinks OpenAI should "throw the baby out with the bath water" and replace, not fork, WebRTC with something that has browser support.

Discord has "forked WebRTC so hard" that native clients implement only a tiny fraction of the protocol — no SDP/ICE/STUN/TURN/DTLS/SCTP/SRTP — but they must still implement everything for web clients.

### But What Instead?

For Voice AI, he'd "start by stream audio over WebSockets" — leveraging existing TCP/HTTP infrastructure instead of building a custom WebRTC load balancer. "It makes for a boring blog post" but is simple, works with Kubernetes, and scales.

He believes "head-of-line blocking is a desirable user experience, not a liability." When packet prioritization is eventually needed, he says OpenAI should copy MoQ and use WebTransport, because "QUIC FIXES THIS."

QUIC connection establishment: "1 for QUIC+TLS" — a dramatic reduction from WebRTC's 8 RTTs.

### Connection ID

QUIC uses a CONNECTION_ID (0–20 bytes) chosen by the receiver, copied from the idea in RFC9146. A QUIC server generates a unique ID per connection, enabling single-port operation while detecting source IP/port changes. When the address changes, "QUIC automatically switches to the new address instead of severing the connection like TCP."

### Stateless Load Balancing

OpenAI's load balancers use a Redis instance to store source IP/port → backend mappings. The author notes: "do you know what is even simpler and easier? Not having a database."

QUIC-LB approach: When a client initiates a connection, the load balancer forwards to a healthy backend. That server completes the handshake and "encodes its own ID into the CONNECTION_ID." Every subsequent QUIC packet contains that backend ID. Load balancers need only "decode the first few bytes and forward it." "Zero state also means zero global state" — these could listen on a global anycast address.

He notes AWS NLB offers QUIC load balancing via QUIC-LB and calls for other cloud providers to catch up.

### Anycast + Unicast

OpenAI appears to assign connections to regional load balancers — "Functional but lame." The author prefers anycast.

Using QUIC's preferred_address: thousands of backend servers worldwide advertise a shared anycast address (e.g., 1.2.3.4). Magic internet routers forward the handshake to one server. Then the server provides its unique unicast address (e.g., 5.6.7.8) via preferred_address.

When a server is overloaded, it stops advertising the anycast address — existing connections remain safe on unicast. "Just like that, no load balancers needed!" The anycast address "is basically a health check."

### Summary

WebRTC:
1. "hurts your product"
2. "hurts your load balancing"
3. "hurts your dog, maybe"

QUIC:
1. "loves your product"
2. "loves your load balancing"
3. "loves your dog, definitely"

He labels QUIC "the chad" and therefore "the superior protocol."

### To Be Fair

He acknowledges OpenAI engineers are "extremely bright" and "dealing with unprecedented levels of stress" — they must scale now. He frames himself as "just some guy who quit my job to work on a passion project" who "literally spend[s] time tracing memes."

Core position: "I just don't think the obvious solution is a good fit for Voice AI." He analogizes: "WebRTC is Jared Leto. There I said it."

He concedes MoQ isn't a perfect fit for Voice AI either — cache/fanout semantics are useless for 1:1 audio. But "You should definitely use QUIC though."
