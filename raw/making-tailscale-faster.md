---
url: https://tailscale.com/blog/making-tailscale-faster
date_fetched: 2026-09-25
---

If you’ve been following Tailscale at all, you know we’re really just a bunch of geeks who care a lot about internet connectivity. One thing we love to talk about is NAT Traversal. That’s one of the core value-adds with Tailscale: we tamed NAT. Not every network is friendly, but Tailscale can still find a path in a wide range of conditions. That’s not the only important thing for an internet protocol: the data plane also has to be performant.

Over the years we’ve been investing in making Tailscale fast. We started by increasing TCP throughput on Linux devices. Then we made significant breakthroughs in wireguard-go to surpass 10Gb/s on bare metal. We later leveraged segmentation offloads to increase throughput over 4x for UDP-based applications. Alongside these improvements to our data plane, we built primitives like Tailscale Peer Relays, which can improve network performance in tricky conditions.

All of this has made Tailscale practical for more performance-sensitive workloads. It means you can use Tailscale for continuous integration, agentic workflows, remote development environments, robotic edge devices, heavy data and telemetry workloads, and more. Tailscale helps those devices connect across a wide range of network conditions.

So yeah, we think Tailscale is fast. But we also think we can make it faster.

Today we’ll detail how we’re boosting throughput for app connectors, subnet routers, and exit nodes, with some multi-queue technology (landing in the second half of 2026). We’ll also preview some throughput and memory overhead improvements we’re deploying in upcoming stable client releases. And we’ll look at some performance tooling issues we want to solve for our customers.

## Less memory overhead for small packets

Most network packets are tiny, like 1 KiB. But to use Linux’s most efficient throughput tools, like Generic Receive Offload (GRO), Tailscale has to be ready to accept 64 KiB of traffic at once. It’s a bit like container shipping: the ports, ships, and trucks are built for one container shape, however full it happens to be.

Tailscale has to unpack those containers—every packet gets decrypted and delivered on its own. The wireguard-go implementation that informs Tailscale’s cryptography and networking essentials, only offers one 64 KiB buffer size to unpack into. So a 1 KiB packet is copied into its own 64 KiB buffer, every time. That’s a rich optimization target.

On Linux and Android, Tailscale now leaves those packets where they landed. It identifies where each one starts and ends inside the single large read instead of copying it somewhere new. Small packets stay small in memory, many share one allocation, and they spend less time being copied. In itself, this led to a roughly 5% speed-up in many network configurations.

Separately, we shortened packet queues—the lines packets wait in between stages of the pipeline. The queues are there to absorb bursts of traffic. Testing showed that most of that depth went unused, while shorter queues meant less waiting time and less memory overhead.

What do we do with all that freed-up memory space? We passed the savings on to some of the hardest-working nodes: subnet routers and app connectors.

## Multi-queue for subnet routers, app connectors, and exit nodes

Subnet routers can look completely different across different tailnets. For someone running a small homelab network, a subnet router can easily handle a small set of `192.168.x.y` non-Tailscale devices. A subnet router that fronts a cloud deployment, one with hundreds of peers, will carry substantially more traffic.

Until recently, subnet routers, app connectors, and exit nodes processed packets for multiple independent streams in one ordered, single-thread pipeline. That meant a single lane was shared across many connections, because a receiving application must never see its own packets arrive out of order.

Having reduced our memory footprint, we had capacity to implement a multi-queue system: several lanes instead of one, scaled to the machine’s resources rather than the number of peers. Each stream of packets gets a lane and stays there, while the lanes run in parallel, allowing work to spread across CPU cores.

It results in higher aggregate capacity and lower delay between receiving and forwarding packets for subnet routers and app connectors. Hardware you already have gets used more efficiently. App connectors and exit nodes, typically serving many users with short-lived connections, get a particularly noticeable boost.

“This translates into lower latency, essentially faster processing of data from the moment we read it off the wire to the moment we send it to the OS,” said Alex Valiushko, member of technical staff at Tailscale.

## Throughput gains with `writev`

Taking advantage of Linux’s `writev` capabilities in the Tailscale client, Tailscale can pass multiple pieces of packet data to the Linux kernel in one operation, rather than having to copy and combine those pieces before passing them to the kernel. The v in `writev` stands for “vector”: Tailscale can describe separate pieces of data that need to be moved, without moving them. It means fewer copies of packet data in memory, fewer write operations, and higher throughput.

## Faster startup with netmap caching

For now, these speed-ups are available only on Linux and, where applicable, Android systems. But we’ve also been working on features that apply to other systems. Tailscale clients will soon be able to use **netmap caching** to start more quickly in many conditions.

A machine connecting to Tailscale usually starts by connecting to Tailscale’s control plane, in something like 100 milliseconds on a typical network. The machine authenticates and gets a "network map" (netmap) describing the devices it can reach and how to reach them. This startup process should feel fast, maybe instantaneous, and with a good network connection, it typically does.

But when you’re on bad airplane Wi-Fi, or inside a hotel with aggressive filtering, or other not-great connectivity setups, it can take a while for the machine to reach the control plane—and sometimes you may not be able to reach it at all. It’s often not obvious where the problem is, but the effect is that you can’t reach other devices.

Even under ideal network conditions, 100 milliseconds of startup latency may be too much for some latency-sensitive workloads.

Netmap caching helps machines get connected when the control plane is not quickly reachable. When it’s enabled, each device on your tailnet stores a copy of the netmap on disk. When a device starts up, it can use that cached copy to establish connections with other devices on the tailnet, until it’s able to contact the control plane to get the latest info. (These connections are negotiated between the devices directly, and Tailscale does not see any of the traffic, as usual).

“Bad network conditions—that’s really the space where people can get a lot of utility out of netmap caching,” said Claus Lensbøl, member of technical staff. “[A device client says], ‘You know what? We haven’t talked to control yet. We’ll probably get there soon. In the meantime, you can still start doing something.’”

There are a few limitations. Caching can only work if the device has previously connected to the tailnet at least once, to fetch a network map from the control plane. In addition, netmap caching requires the device to have persistent disk space to store the cache. We’ve taken care to minimize unnecessary disk writes, but in some cases you may not want to enable it. For example, on exceptionally large tailnets, updating a cache may require a lot of disk traffic. Likewise, devices that use slow or wear-sensitive storage like SD cards may prefer not to enable netmap caching.

For most devices on most tailnets, though, this feature can notably speed up how quickly devices can establish contact with each other at startup. We’ve seen tailnets with poor control plane reachability start sending through the data plane, on a “warm” cache start, one to two orders of magnitude faster than from a “cold” start. For devices facing variable startup latency, or far away from a DERP server or the control plane, the benefits are particularly tangible.

## When you can see all these speed-ups

- **Memory reduction**via buffer changes (Linux/Android) is expected in the v1.104 client.
- **Multi-queue**to benefit subnet routers and app connectors is planned for a release after v1.104.
- **Throughput gains**(Linux/Android) were partially implemented in spring 2026; leveraging the additional gains in memory and throughput is planned for a release after v1.104.
- **Netmap caching**is available as a feature flag in the current Tailscale client; it is expected to arrive by default in v1.104, following further testing. Mobile clients are expected to have the feature in a release after v1.104.

## Performance is still difficult to diagnose and test

Sure, we think Tailscale is fast. But you shouldn’t have to trust us on that. That’s why we’re exploring a Tailscale-aware monitoring and testing toolkit. We want to give our customers the tooling they need to test, diagnose, and understand their network configuration, in a way that’s Tailscale-native.

Here are the gaps we see in modern performance testing:

- **Distribution tax:**Most performance tooling is point-to-point, and requires you to install something on every endpoint.
- **Workflows are rigid:**It’s pretty easy to run the wrong test, get the wrong output, and chase a problem that’s not there.
- **Protocol support:**Many tools don’t support newer protocols, such as QUIC and HTTP/3.
- **Tailscale-awareness:**General-purpose tooling is not Tailscale-native. It can’t tell you if a connection is using DERP or is direct, whether a peer relay might help, or how the connection path changes over time.

Existing tooling doesn’t understand Tailscale-native paths and states. So we’re exploring tooling that does. Help us shape the future of performance testing at Tailscale.
