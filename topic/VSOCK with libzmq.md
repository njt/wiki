# VSOCK with libzmq

How to get ZeroMQ's messaging patterns (req/rep, pub/sub) and CurveZMQ authentication onto Linux's AF_VSOCK transport — the socket family for guest↔hypervisor and guest↔guest communication — before VSOCK support lands in a stable libzmq release. Rémi Jouannet's write-up of the PR (#4822) that adapted libzmq's existing VMCI transport into a VSOCK transport, plus his pyzmq fork (pyzmq-vsock) and the loopback-CID trick that lets you test it all without a VM.

---

## Key Quotes

> "If you are planning to use vsock, you'll quickly notice that you only have low-level bindings available, there is no magical library that abstracts the socket for you."

The honest statement of the gap. VSOCK's bindings hand you a raw socket and leave you to re-derive poll/recv/send/listen/bind — the same socket plumbing libzmq exists to eliminate. This is the article's whole justification in one line.

> "libzmq does not make regular releases because it is stable and does not move that much. It is also a library made for embedded software it's easy to build it statically over a specific commit."

The reason the VSOCK support isn't generally available yet, and a tacit acknowledgment that "use this feature" means "build from a commit or use my fork." The fork-as-distribution-channel move — ship a prebuilt pyzmq wheel against a pinned libzmq commit — is the pragmatic bridge over upstream's glacial release cadence.

> "Since Linux 5.6 there is a loopback CID (VMADDR_CID_LOCAL) so you can test VSOCK without running a VM."

Small but load-bearing. `vsock://@:5555` makes the whole thing testable on a laptop, which is what turns a niche VM feature into something a developer can actually try in five minutes.

---

## Key Themes

#concept #tool #pattern

**VSOCK as a transport, not a product.** AF_VSOCK is a kernel address family (in-tree since 4.8, VMware's VMCI reborn) with CIDs instead of IPs, ports, and stream/datagram modes. It's the native channel between a guest and its hypervisor — the same channel [[Stockyard]] uses to trigger ZFS snapshots from inside a Firecracker VM, only here it's exposed as a general-purpose socket.

**A messaging library over an exotic socket.** libzmq's pitch is that it implements basic patterns and good practices "over almost any kind of socket." Adding VSOCK means req/rep and pub/sub — and CurveZMQ auth — become available on the guest↔host channel in every language with a libzmq binding. The PR was "almost just copied and pasted from VMCI," which is the quiet lesson: the transport abstraction already existed, and VSOCK was a small delta on top of it.

**Loopback CID as the test seam.** `VMADDR_CID_LOCAL` (Linux 5.6) plus the `@` shorthand in ZMQ URIs removes the VM requirement from the development loop — the same reason [[Stockyard]]'s audit trail is demoable, applied to messaging.

---

## Critical Analysis

The article is a clean, narrow engineering write-up and is better for that. It doesn't oversell: it names the exact PR, the exact fork, and the exact command to reproduce the result, and it shows both the Curve success path and the denied-key failure path — a rare, welcome honesty in a field of hello-world-only demos.

The most interesting claim is not technical but distributional. VSOCK support is *done* (merged upstream), yet the practical answer to "how do I use it" is still "install a fork from a third-party GitHub release." That's a real signal about the cost of upstream's stability-first release policy: a merged feature that nobody can consume is functionally a missing feature until someone ships a wheel. The fork (pyzmq-vsock) is the unsung infrastructure hero of the piece.

The security angle is understated but worth pulling out. CurveZMQ gives you authenticated, encrypted messaging over a channel that is otherwise just a raw socket between guest and host — and the guest↔host boundary is precisely where you don't want unauthenticated traffic. For the agent-sandboxing world this wiki tracks, that's the useful takeaway: VSOCK is how [[Stockyard]] and [[Security and Sandboxing]]'s micro-VM world signals across the isolation boundary, and libzmq's Curve transport is one answer to securing that channel. The article gestures at this ("security features") but stops at the demo; the production threat model (who else can bind a CID/port pair) is left as an exercise.

The honest caveat: everything here is Linux-specific, and the article assumes the reader already knows why they want guest↔host messaging in the first place. As a reference for "how do I actually do VSOCK messaging today" it's excellent; as an argument for *why* you'd choose VSOCK over, say, a virtio serial port or plain AF_UNIX, it defers to the man pages and Proxmox's qemu-guest-agent.

---

*Sources: [[raw/libzmq-vsock-pyzmq]], [[summary/libzmq-vsock-pyzmq]]*
*Last updated: 2026-09-11*
