---
url: https://blog.apnic.net/2025/12/08/21-years-and-counting-of-eight-fallacies-of-distributed-computing/
title: "21 years and counting of 'eight fallacies of distributed computing'"
author: George Michaelson
date_fetched: 2026-06-15
date_published: 2025-12-08
---

# 21 years and counting of 'eight fallacies of distributed computing'

**Author:** George Michaelson
**Published:** 2025-12-08, APNIC Blog

## Summary

Michaelson revisits the eight fallacies of distributed computing — a list that originated at Sun Microsystems from Bill Joy, Tom Lyon, L. Peter Deutsch, and James Gosling — and examines how each holds up in the modern Internet. Despite decades of accumulated networking knowledge, developers continue to design systems as if these fallacies weren't real. The piece walks through each fallacy with concrete modern examples: how Wi-Fi bottlenecks contradict "bandwidth is infinite," how BGP topology changes break assumptions about static networks, and how traffic analysis undermines even encrypted connections.

## The Eight Fallacies

1. The network is reliable
2. Latency is zero
3. Bandwidth is infinite
4. The network is secure
5. Topology doesn't change
6. There is one administrator
7. Transport cost is zero
8. The network is homogeneous

## Origins

The first four were collected by Bill Joy and Tom Lyon at Sun Microsystems. L. Peter Deutsch added three more while at Sun, and James Gosling contributed the eighth. Michaelson notes that Sun's integration of high-speed graphics, UNIX, and Internet protocols "led to the explosion in desktop computing" and credits Sun with ZFS, NFS, and Java.

The list was aimed at network software developers — people writing code that had to handle the reality that packets get lost, networks change, and nothing is free.

## Detailed Analysis

### 1. The Network Is Reliable

The Internet is "probably broken somewhere, for some users, at all times." IP does not guarantee delivery. Higher layers like TCP and QUIC handle loss detection through retransmission. The three classic measures: loss, delay, and jitter.

### 2. Latency Is Zero

Delay stems partly from physical distance — "the speed of light in fibre is slower than in a vacuum." Jitter (delay variability) is particularly challenging for gaming and streaming. Services like Netflix use buffering and forward error correction to compensate, but the fundamental physics remain.

### 3. Bandwidth Is Infinite

Many links carry "more people sending packets than there are spaces available." Finite bandwidth creates queuing, which causes delay and jitter, and under extreme conditions, packet loss. Home Wi-Fi is a common bottleneck — a modern phone may sustain 400 Mbit/s while a five-year-old router caps at 100 Mbit/s.

### 4. The Network Is Secure

In the telecom monopoly era, one provider ran entire networks end-to-end. Today, "networks run across multiple providers and through intermediaries." Even encrypted packets leak information through traffic analysis: "never rely on it below your HTTPS or TLS connections to hide you."

### 5. Topology Doesn't Change

Changes occur when phones switch towers or providers reroute traffic. Protocols like QUIC and TCP mitigate disruption, but the overhead processes — VRRP, CARP, BGP, Multipath TCP — are not cost-free.

### 6. There Is One Administrator

"Sometimes, it feels as if there isn't even a single administrator." The person you speak to about a network problem is often not the person making changes. Modern network complexity means no single entity has a complete view.

### 7. Transport Cost Is Zero

Cost spans electricity, hardware, support systems, and business logic. Michaelson uses SMS as a stark example: sending a movie via SMS would cost thousands of dollars. "Just because cost is not exposed directly in a protocol does not mean it doesn't exist."

### 8. The Network Is Homogeneous

Visible in BGP routing where AS path length is mistaken for true cost. At home, Wi-Fi and Ethernet devices compete differently for the same medium. "IP masks many of the nuances between local and remote, slow and fast."

## Meta-Fallacy

Michaelson notes a recurring error: online discussions "mistakenly refer to Tom Lyon as Dave Lyon." Even the facts about the fallacies themselves can't be counted on to remain fixed. He speculates the list might grow to nine or ten but "is unlikely to drop down to seven."
