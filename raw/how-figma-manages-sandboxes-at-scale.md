---
url: https://www.figma.com/blog/how-figma-manages-sandboxes-at-scale/
date_fetched: 2026-11-06
---

# How Figma manages sandboxes at scale

Figma’s AI products require tens of thousands of sandboxes to run untrusted code safely. That’s why we built Nimbus: a platform that makes sandboxes fast, reliable, and easy for Figma product teams and operators to manage.

## Share How Figma manages sandboxes at scale

Illustrations by Pete Gamlen

Figma users are moving faster than ever with AI tools like the agent Starting today, work with an agent that is built for Figma—directly on the canvas. Today we’re introducing Figma Make, a new prompt-to-app capability to help you quickly explore, iterate, and refine—whether it's generating high-fidelity prototypes or getting into the details in design and code. In this three-part series, our security engineering team shares practical tips for deploying and operating application sandboxing techniques. First up: evaluating the many sandboxing options, and how to think about the trade-offs between them.## The Figma design agent is here

## Introducing Figma Make: A new way to test, edit, and prompt designs

## Server-side sandboxing: An introduction


Late last year, as Figma was scoping and shipping more AI features, we realized we’d soon be managing tens of thousands of sandboxes across multiple Figma products, all with different memory and network permission requirements. We knew we’d need a way to address these complexities at scale.

So we built a centralized platform for managing sandboxes, which we call Nimbus. With a simple API interface, Nimbus makes it easy for any engineer at Figma to create, reuse, monitor, and delete sandboxes. It handles all the provider-specific details, like resource allocation and network security, so our product teams don’t have to, and gives us the flexibility to add new providers (or homegrown infrastructure) without requiring teams to change their workflows.

Here’s how Nimbus works, and what it makes possible at Figma.

## How Nimbus works

To request a sandbox, a Figma product—the agent, for example—provides a configuration file that we call a namespace. Given a namespace, the Nimbus control plane selects a provider, allocates the specified resources, applies the relevant networking and security policies, and manages the sandbox lifecycle.

Nimbus provisions each sandbox using `sandbox-base`, a base image that gives every sandbox the same starting environment across providers. Inside each sandbox, Nimbus also runs `sandboxd`, a platform daemon that starts and supervises workload processes, manages network listeners and authentication, runs health checks, reports metrics, and handles shutdown. Because it owns the process tree, `sandboxd` also provides observability into a sandbox’s CPU, memory, and network usage, helping teams monitor resource consumption and diagnose unexpected failures, like out-of-memory crashes.

We designed Nimbus with the following requirements in mind:

- **Developer experience:**We wanted Figma engineers to be able to treat a sandbox request as a configuration task, instead of a request to fork a service or copy a provider integration. This would ensure that product teams didn’t have to learn provider-specific nuances or reinvent the wheel to work with sandboxes.
- **Isolation and security:**Workloads needed to adhere to security policies limiting what they could access and how they would communicate. The security guarantees had to be the same regardless of the provider.
- **Performance and reliability:**Sandboxes had to start quickly—ideally, by the time a user began interacting with a feature—and our goal was for Nimbus to return a provisioned sandbox in under one second. Nimbus also needed to load-balance requests while accounting for provider health, so product teams wouldn’t have to worry about cloud provider failures.

## A contract between the product and the platform

Nimbus’s namespace design is similar to the Kubernetes operator pattern, where the product declares its intent while the control plane owns the work required to support it.

With a namespace, teams can describe what kind of sandbox they want, and Nimbus will handle the rest.

The namespace specifies the container image and resources required, along with additional health and security controls. It can also set boundaries for how many provisioned sandbox instances Nimbus should keep on hand.

```
{
  "name": "example-workload",
  "imageRepository": "url://",
  "services": {
    "ninja": {
      "command": ["/app/server"],
	"port": 1000
    }
  },
  "healthCheck": {
    "http": {
      "portName": "ninja",
      "path": "/healthz"
    }
  },
  "resources": {
    "cpu": "2",
    "memory": "1Gi"
  },
  "warmPool": {
    "minSize": 10,
    "maxSize": 100
  },
  "envVars": {
    "logLevel": "debug"	
   },
  "allowedHosts": [
     "*.figma.com"   
   ]
}
```
The namespace creates a clear separation of responsibilities. Product teams can choose the workload behavior, and the platform chooses how to apply security rules, distribute resources, report outcomes, and more. Sandbox behavior is consistent across teams, and it’s much easier for us to iterate on our sandbox architecture. We can change how we balance or evolve providers under the hood without impacting the teams using sandboxes.

In addition to the namespace, Nimbus exposes three APIs for the sandbox lifecycle: `GetOrCreate`, `Get`, and `Delete`.

- `GetOrCreate`takes a namespace and a custom key and returns the same sandbox for repeated requests with that key, creating one if necessary.
- `Get`retrieves the sandbox and its current lifecycle state.
- `Delete`tears down the sandbox.

While the API is simple, the work that Nimbus does to fulfill its end of the contract is anything but. A `GetOrCreate` request can’t return merely a running container; it has to return a sandbox that satisfies the namespace quickly and securely, regardless of the provider.

## A container boundary isn’t enough

A sandbox isolates AI-generated or otherwise untrusted code from other processes running on Figma servers. But from a security perspective, that’s not enough. The code could still send information across the network that we don’t want to leave the sandbox. The naive solution would have been to implement a strict network allowlist, but that’s often too blunt for the products we want to build. AI workloads may need to search and read arbitrary pages on the internet, which an allowlist would prevent.

As we built Nimbus, we landed on a solution that treats reads differently from writes. Through the namespace, product teams can explicitly allow a list of hosts that are completely trusted—the code can read data from them or send them data. For destinations outside that list, Nimbus permits *only* read requests—`GET`, `HEAD`, or `OPTIONS`—with no request body. That’s enough for features like web search and page retrieval, while `POST`, `PUT`, or any request with a body to arbitrary endpoints remains blocked. Product teams can also dynamically update the policy for a running sandbox if necessary.

We need to ensure that switching between providers doesn’t change the sandbox’s runtime behavior or security posture. To do that, we regularly test all providers against the same expectations for networking, credentials, lifecycle behavior, observability, and failure handling.

If a provider has built-in support for inspecting and controlling outbound traffic, Nimbus installs the policy there. On other providers, Nimbus attaches a proxy sidecar—a second container that runs alongside the workload—and routes outbound HTTPS traffic through it to evaluate each request. In short: We enforce the policy consistently regardless of the controls the provider gives us.

Nimbus doesn’t let the request go through if a sidecar can’t be attached, a policy can’t be evaluated, or a credential can’t be decrypted.

We also needed to think about how workloads get access to credentials. Nimbus encrypts credentials before they enter the sandbox, and swaps the encrypted value for the real credential immediately before the request leaves the sandbox—and *only* when it’s being sent to its intended destination. If untrusted code tries to send the credential somewhere else, all it has access to is the encrypted value.

As a result, because all workloads use the same security requirements, Nimbus can prepare sandboxes ahead of demand.

## Warm is fast—and also complex

Rather than creating every sandbox when a request comes in, Nimbus maintains a pool of already-running, unassigned, “warm” sandboxes for each namespace. Warm pools cut our provisioning latency by 90% compared to spinning up a new sandbox each time a request comes in. Because the warm pool stocks up minutes’ worth of sandboxes, it absorbs those minutes of provider outages. Plus, Nimbus’s provisioning availability increased from 99% to 99.95% as a result of the warm pool and automated provider selection.

To make warm pools work as intended, however, Nimbus has to keep the right number of healthy sandboxes available as demand and provider conditions change.

### Claiming a sandbox exactly once

For a given namespace, multiple Nimbus pods handle sandbox requests concurrently. Since every pod can draw from the same pool, Nimbus needs to ensure that two pods can’t claim the same sandbox, without forcing every consumer to wait behind a global lock.

Nimbus handles this in Postgres. Each claim is a single transaction: Select the oldest matching sandbox, skip any row another request is already claiming, remove the chosen row from the pool, and return its connection details.

### Sizing the pool from demand

A fixed target is easy to reason about; but at Figma’s scale, that’s impractical. We’d either have a target that’s too low and run out of capacity during a peak, or a target that’s too high and spend money on unused sandboxes. Instead, Nimbus watches how quickly sandboxes are being claimed and dynamically resizes the pool, while respecting lower and upper bounds set by the namespace.

### Routing around provider degradation

Nimbus starts with configured weights for each provider, then adjusts how much work each receives based on recent provisioning outcomes. If a provider slows down or starts returning errors, Nimbus shifts work toward healthier providers.

We’re careful about which failures feed into these decisions. A client cancellation isn’t evidence of a provider outage, for example, and rate limiting imposed by Nimbus itself shouldn’t trip a provider circuit breaker. We also treat ambiguous timeouts differently from confirmed failures.

### Ensuring sandboxes are healthy

Nimbus continually checks the health of every sandbox, including warm capacity that hasn’t been assigned yet. A Redis-backed queue tracks when each sandbox is due for its next check.

Nimbus won’t destroy a potentially recoverable sandbox after a single failed check or an ambiguous provider response. It waits for repeated failures before marking the sandbox unhealthy.

During each check, Nimbus does the following:

- Tries to reach `sandboxd`. If it responds, the container is healthy—it’s reachable, and the Nimbus runtime inside it is functioning.
- If `sandboxd`doesn’t respond, Nimbus falls back to the provider’s view of that sandbox.
- A provider-confirmed termination is definitive, so Nimbus marks the sandbox unhealthy. If that sandbox is still sitting in the warm pool, Nimbus first claims its pool entry. This avoids the problem of a request receiving a sandbox that’s in the process of being deleted.

### Testing the control loop

Together, the components above form a control loop: Nimbus repeatedly checks the state of each warm pool and takes action to keep the right number of healthy sandboxes available.

We built a simulator that runs the same control loop logic against the conditions we expect in production. We can simulate wiping a pool during a deployment, aging many sandboxes out at once, or degrading multiple providers together, then test whether the controller recovers.

This testing has helped us fine-tune our logic and uncover issues that are otherwise hard to detect. For example, we originally designed our routing signals to primarily react to provider failures, so a provisioning request that took much longer than usual yet eventually succeeded still counted as a success. But after dispatching a batch of provisioning requests, Nimbus would wait for the entire batch to finish before checking the warm pool again. If a provider slowed down, new requests could continue draining the pool while Nimbus waited. When the batch eventually completed, the provider could still appear healthy because its requests had succeeded. Meanwhile, the pool deficit would grow, so Nimbus could end up sending another even larger batch to the same slow provider.

With the simulator’s help, we identified and addressed both sides of this particular problem. Slow provisioning requests now reduce a provider’s effective weight even when they eventually succeed, allowing Nimbus to shift new work toward faster providers. We also moved provisioning out of the control loop: Now, Nimbus dispatches requests and immediately returns to observing the pool rather than waiting for provisioning to finish.

## Where we go next

There’s more work ahead: We’re expanding namespace adoption, improving autoscaling and provider weighting, and continuing to optimize latency and cost. We’re also exploring a way to preserve a sandbox's working directory and restore it into another warm sandbox, even if that sandbox comes from a different provider.

Since launching internally in November 2025, Nimbus has grown into shared infrastructure for multiple product surfaces. Most product teams will never know which path served their sandbox, which provider created it, or which daemon kept its processes healthy—which is what we intended. Because we built so much automation into the platform, incidents are now rarer. Instead of paging an on-call engineer every time a provider has an outage, Nimbus can reroute traffic to a healthy provider automatically.

###### We’re hiring engineers!

Learn more about life at Figma, and browse our open roles.

Finally, because platform capabilities live in Nimbus rather than any one provider, we can continue evolving the infrastructure without changing the interface our products build against. That gives us a foundation we can keep building on as more AI-powered experiences at Figma need safe, interactive compute.

*This work is the product of countless contributions by the Sandbox Platform team in collaboration with many other engineers across the Figma Infrastructure team. It was made possible by Bashar Al-Rawi, Brandon Guo, Carl He, Chen Luo, David Mather, Ian VonSeggern, Tristan Schneiter, and Yong Su Lee.*

Pratik is a software engineer on Figma’s Sandbox Platform team and a founding engineer on Nimbus, supporting Figma’s AI infrastructure. Previously, he worked on Figma’s Databases team and was a software engineer at Momento and AWS.

Jordan is a software engineer at Figma, where he is a founding engineer on Nimbus and leads the Sandbox Platform team, supporting Figma’s AI infrastructure. He was previously a platform engineer at Mux and an SRE at Slack.
