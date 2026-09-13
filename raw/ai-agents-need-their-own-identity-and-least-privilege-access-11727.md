---
url: https://shiftmag.dev/ai-agents-need-their-own-identity-and-least-privilege-access-11727/
date_fetched: 2026-09-13
---

# AI agents need their own identity and least-privilege access

Infrastructure keeps shifting and access now comes from all kinds of people and systems.

I spoke about that with **Ross Kukulinski** from Tailscale at WAD Berlin. He kept coming back to the same problem: traditional networking still trusts too much based on IPs and subnets. That doesn’t work well in a world where identity matters more than location.

We talked about his talk on the next internet protocol and what it takes to build connectivity around identity instead of network boundaries.

Kukulinski points out a simple mismatch: the internet was built to be open, but private infrastructure usually shouldn’t be. That openness works for public services, but it gets much harder to justify when you’re talking about databases, Kubernetes clusters, internal tools, and AI workloads.

In practice, teams try to work around this with firewalls, gateways, proxies, and network segmentation

Kukulinski argues that a better starting point is to ask whether every person, machine, or workload should be able to reach everything else at all. This shifts the design toward **smaller, isolated networks that are only connected when a resource needs to be shared**, with access rules determining which identities can communicate with which services. He explained:

The key is point-to-point connectivity. Policy is governed centrally, but enforcement happens at the edge. Instead of routing through a stack of gateways or proxies, machines can talk directly to each other.


For developers, the key idea is **the separation of authorization from network topology**. A service can be reachable because a particular identity is allowed to reach it, rather than just because both endpoints sit inside the same trusted network.

“That distinction matters as infrastructure becomes more distributed. A development environment may span several clouds, customer networks, on-prem systems, remote workstations, and edge devices,” Kukulinski pointed out.

He added that traditional connectivity options can solve individual cases, but large shared networks become harder to govern as the number of environments and access relationships grows. The goal is to **make connectivity explicit and narrowly scoped**, instead of treating broad network membership as permission.

## Identity is more durable than an IP address

That same idea raises a bigger problem with trusting IP addresses: IPs are fine for finding where traffic should go, but they are not a good way to prove who someone is or whether they should be allowed to access something sensitive. Kukulinski said this is especially clear in Kubernetes, where IP addresses change often and don’t reliably represent identity:

IP addresses come and go, and they’re duplicated everywhere. If you’re running in a Kubernetes cluster, the IP address of any one pod changes as pods are destroyed and recreated. So trusting IP addresses, or even trusting a subnet, really doesn’t work.


Kukulinski also said that **identity-based access is better because it ties permissions to the connection itself**, rather than to an IP address or subnet. That connection can carry useful context, like who the user is, what group they belong to in the identity system, what device they’re using, and what policies apply to that device. For example, a managed work laptop might be allowed to access internal systems that a personal laptop cannot, even if both belong to the same person.

For developers, **this can remove a lot of the hassle around infrastructure access**. SSH, Kubernetes admin tasks, databases, and remote systems often rely on credentials that have to be created, shared, updated, and eventually revoked.

## Identity is becoming the control layer for infrastructure and AI

Kukulinski also said that if access is based on identity and group membership, **permissions can change automatically when a person’s role changes**. For example, when a developer moves to another team, the systems they can reach can update right away, without having to clean up old VPN access or long-lived keys:

If I transfer internally from product management to engineering, and my groups change in the identity provider, that automatically updates what I can do from my devices.


The same idea is becoming important for AI tools too. An identity-aware gateway can sit between developers and model providers, check who is making the request, control which models they can use, and manage provider logins in one place. As Kukulinski explained, developers can still make API calls without needing a separate credential for every model or service.

He also said **Kubernetes shows that networking can still get complicated**, even when identity is handled well:

The harder problem begins at the cluster boundary, when a service in one cluster must communicate securely with another cluster, another cloud, or a system outside Kubernetes. Things inside the cluster can talk easily, but talking to other Kubernetes clusters, especially in a different cloud, is incredibly challenging


Kukulinski prefers a **direct encrypted connection** when possible, using relay infrastructure only as a backup. Connectivity affects not just security, but also speed, reliability, and cloud costs.

## AI agents need limited access, not blanket trust

AI makes this problem harder because **it adds non-human users that need access to internal systems**. Kukulinski also said AI tools can help attackers automate scanning and reconnaissance, so defenders should use the same kind of automation for supply-chain security, environment checks, and build and deployment controls.

The risk is often inside a company’s own infrastructure. Teams sometimes give AI projects broad access first and tighten it later, which can leave them with the same kind of over-permissive VPN access companies have struggled with for years:

The historical pattern has often been: “We just need to get this AI thing going, so let’s give AI access to the whole network.” That’s the scariest thing, and it repeats the same failure mode as giving too many people open VPN access to too many systems.


Kukulinski’s preferred model is to **give agents their own identity and access policy, with permissions that can be changed or revoked as needed**. An agent, or group of agents, can also be isolated from the wider network and given access only to the nodes, services, or data it needs.

For developers building agentic systems, treat an agent as a separate workload with least-privilege access from the start

“If an agent can reach a production database simply because it is already inside a trusted network, the architecture has repeated an old networking mistake with a much more capable actor,” Kukulinski said.
