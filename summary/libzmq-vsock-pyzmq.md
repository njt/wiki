---
url: https://blog.remijouan.net/posts/libzmq-vsock-pyzmq/
title: "Use VSOCK with libzmq"
author: Rémi Jouannet
date_fetched: 2026-09-11
topics:
  - software-engineering-craft
---

A how-to for using Linux's AF_VSOCK address family through ZeroMQ (libzmq), plus the author's own fork of pyzmq that makes it available before it lands in an official release. VSOCK is the socket family for guest↔hypervisor and guest↔guest communication on the same host — originally VMware's VMCI, in the kernel since 4.8. It looks and feels like a Unix socket with TCP/UDP-style addressing: 32-bit "context identifiers" (CIDs) instead of IPs, plus ports, with stream and datagram modes.

## The gap

VSOCK only ships low-level bindings (Python, C, Go, Rust); there is no library that abstracts poll/recv/send/listen/bind away. libzmq already had a VMCI transport but no VSOCK. The author contributed VSOCK support by adapting the VMCI code — PR #4822. This brings libzmq's higher-level patterns (req/rep, pub/sub) and CurveZMQ authentication to the VSOCK transport, and via libzmq's bindings, to every language that has one (Python, Ruby, Node.js, Perl, Java, Lua…).

## Availability

libzmq releases rarely, so VSOCK support isn't in a stable release yet. The author forked pyzmq (pyzmq-vsock) to build against a recent libzmq commit, publishing a prebuilt wheel. Since Linux 5.6 the loopback CID (`VMADDR_CID_LOCAL`, addressed as `@` in ZMQ URIs) lets you test VSOCK with no VM — with `vsock://@:5555` as the bind address. The post walks through a req/rep "Hello World" pair and a full Curve-authenticated asyncio example (keys, `AsyncioAuthenticator`, and the denied-key output), showing both the success and "server rejected this client key" paths.
