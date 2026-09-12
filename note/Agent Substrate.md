# Agent Substrate

Google's open-source platform for running AI agents at scale on Kubernetes — not an SDK for building agents, but an infrastructure layer for deploying them. Its core insight is that agent workloads are bursty (mostly idle, brief bursts of work), and the standard model of one-pod-per-agent wastes resources. Agent Substrate multiplexes hundreds of agents onto a small pool of pre-warmed worker pods by checkpointing idle actors and restoring them on-demand in under a second.

This is the first open-source attempt at "serverless for stateful agents" — not stateless functions, but long-lived processes with memory and filesystem state that can be hibernated and revived transparently.

---

## Architecture

Agent Substrate is a **seven-component distributed system** layered on Kubernetes:

### Control plane: `ateapi` (cmd/ateapi/main.go, 559 lines)
A gRPC server that owns the actor lifecycle, scheduling, and snapshot coordination. It uses **Redis/ValKey** as its state store (not etcd/kube-apiserver) because it needs to handle millions of actor records with sub-100ms read-modify-write cycles — the Kubernetes API server is not designed for this. The store interface (`cmd/ateapi/internal/store/store.go`) defines optimistic concurrency (version-checked updates), distributed locks, and worker watches — essentially a lightweight distributed database on Redis.

The scheduler (`cmd/ateapi/internal/scheduling/scheduling.go`) is deliberately **simple**: filter workers by sandbox class, template/actor label selectors, and node affinity, then pick randomly from candidates. No bin-packing, no cost-model optimization — just "find a free worker, any worker." The design bets that oversubscription (30:1 or higher) makes sophisticated scheduling unnecessary; workers are fungible.

### Kubernetes controller: `atecontroller` (cmd/atecontroller/main.go, 192 lines)
Standard controller-runtime reconcilers for three CRDs: `WorkerPool` → `Deployment`, `ActorTemplate` → golden snapshot creation, and `NetworkPolicy` enforcement. The controller is thin by design — it handles the slow, declarative infrastructure lifecycle (creating pods, updating deployments) while the control plane handles the fast, imperative actor lifecycle.

### Node supervisor: `atelet` (cmd/atelet/main.go, 1,551 lines)
The **largest component**, running as a DaemonSet on every node. Its responsibilities:
1. **Image management**: pulls OCI images into a node-local cache (`internal/imagecache/`), unpacking layers to disk so gVisor's `runsc` can use them
2. **Snapshot I/O**: streams checkpoint images to/from GCS or S3, with zstd compression
3. **Volume mounting**: CSI volume setup/teardown per actor
4. **Credential brokering**: a Unix socket proxy that gives worker pods short-lived JWTs for actor identity without baking credentials into container images
5. **ateom communication**: dials the ateom gRPC socket inside each worker pod to trigger Run/Checkpoint/Restore

The atelet's `Checkpoint` and `Restore` methods (`main.go:490-947`) are the heart of the system, with **instrumented phase timing** for every step (sandbox assets, ateom checkpoint, GCS upload, etc.) emitted as OpenTelemetry metrics.

### Sandbox herder: `ateom` (cmd/ateom-gvisor/main.go, 918 lines; cmd/ateom-microvm/main.go, 420 lines)
Runs inside each worker pod, one per sandbox class. It's the **bridge between the Kubernetes world and the sandbox runtime**:
- **gVisor**: drives `runsc` commands for create, checkpoint, restore, and delete. Sets up cgroup delegation, a dedicated network namespace the sandbox reads/resets, and an `atunnel` sidecar for actor ingress proxying
- **micro-VM**: drives Cloud Hypervisor for VM lifecycle, with `userfaultfd`-based memory demand-paging during restore (the VM boots immediately, pages fault in on access), `tmpfs` overlay for rootfs writes, and virtio-fs for `DurableDir` volumes

The key design separation: the ateom **owns the pod's lifetime but not the actor's** — the actor comes and goes within a persistent pod, which is how workers stay warm.

### Networking: `atenet` (cmd/atenet/main.go, 21 lines + Envoy ext_proc)
A lightweight networking stack. The `atenet-router` runs Envoy with an external processor that:
1. Receives HTTP traffic addressed to `<actor>.<atespace>.actors.resources.substrate.ate.dev`
2. Calls `ateapi.ResumeActor()` if the actor is suspended
3. Opens an authenticated TLS tunnel to the worker's `atunnel` sidecar on port 443
4. Forwards the original request

This is the **wakeup-on-demand mechanism**: traffic itself triggers resumption, so suspended actors appear always-available.

### CLI: `kubectl-ate` (cmd/kubectl-ate/main.go, 23 lines)
A kubectl plugin wrapping the gRPC API for interactive management.

---

## Key Techniques

### Checkpoint/restore as the core primitive
Unlike Docker pause/unpause (which keeps the process in memory), Agent Substrate's checkpoint/restore **fully frees worker resources**: the actor's process memory and filesystem are captured to a snapshot, the worker sandbox is torn down, and the worker returns to the pool. On resume, the snapshot is restored into any available worker. This is how 250+ actors run on 8 physical pods in the demo — each actor only occupies a worker while actively processing a request.

### Self-describing snapshot manifests
Every snapshot includes a JSON manifest (`sandboxManifestName`) recording exactly which sandbox binary versions created it. This solves a reproducibility problem: if you checkpoint with `runsc` v1.2.3 and later the cluster upgrades to v1.3.0, the restore pulls the pinned binary version from the manifest, not whatever is current. The manifest also lists exactly which files the runtime wrote (gVisor image files, Cloud Hypervisor state files), so the system ships precisely that set rather than a hardcoded file list.

### Concurrent restore pipeline
During restore (`atelet/main.go:812-888`), the GCS snapshot download and the OCI image unpack run in **parallel via `errgroup`**. These are independent — only the final `ateom.RestoreWorkload` needs both — so overlapping them hides whichever leg is shorter. On a cold node the overlap is substantial (~2.5s image unpack masked behind the download).

### Golden snapshots + DATA_ON_GOLDEN
When an `ActorTemplate` is created, the system boots the workload once and captures a "golden snapshot" — a shared, versioned baseline. New actors restore from this golden snapshot instead of cold-booting, giving near-instant first activation. For micro-VMs, the `DATA_ON_GOLDEN` scope takes this further: the actor's snapshot captures only durable data (a tar of `DurableDir` volumes), and on resume that data is served to the golden's still-warm guest state. The golden's **memory image must be resumed by the exact binary that created it**, so the manifest from the golden (not the actor) supplies the runtime binaries.

### Sandbox runtime as runtime-fetched asset
The gVisor `runsc` binary and micro-VM kernel/firmware are not baked into worker images. They're defined in a cluster-scoped `SandboxConfig` CRD and fetched at runtime by the atelet. Each snapshot's manifest pins the version used, so restores are reproducible even as the cluster's default sandbox binaries are upgraded. This also means one `SandboxConfig` pins the runtime for many templates.

### Distributed lock-based worker assignment
When an actor resumes, the control plane acquires a distributed lock (Redis-backed) on the worker, assigns the actor, and releases the lock. This prevents double-assignment without requiring a single-threaded scheduler. The lock is auto-renewed; if renewal fails, the lease context is cancelled.

### Crash classification
Actor crashes are classified (`internal/ateerrors/`) with structured reasons (`ReasonInvalidSandboxAsset`, `ReasonTerminalFileSystemError`, etc.) that determine whether the actor enters a CRASHED state (unrecoverable) or can be retried. This prevents a bad image from triggering infinite restart loops across the cluster.

---

## Design Decisions

### Build a custom control plane rather than extend Kubernetes
**The trade-off**: running a separate control plane (ateapi + Redis) adds operational complexity, but the Kubernetes API server is structurally unsuited for millions of high-frequency reads/writes. Kubernetes reconciles resources asynchronously across controllers with eventual consistency; Agent Substrate needs synchronous, atomic worker assignment with sub-100ms latency. The Redis store gives them the performance profile they need, at the cost of managing another stateful service.

### Keep the scheduler simple
The scheduler (`scheduling.go`) is ~140 lines of random selection among matching workers. There's no cost model, no bin-packing, no predictive placement based on snapshot locality (yet — the architecture doc flags data locality as a future concern). The bet: at 30:1 oversubscription, sophisticated scheduling adds latency without meaningfully improving utilization. The `RequiredNodes` constraint (for local snapshots) is the only localization concern.

### Snapshots are version-locked to sandbox binaries
This is a conservative choice that ensures correctness at the cost of operational complexity (you can't transparently upgrade runsc for running actors). The alternative — decoupling snapshot format from runtime version — would be faster to upgrade but risks subtle incompatibility bugs in checkpoint/restore across versions.

### Atespace as first-class isolation concept
Rather than mapping actors 1:1 to Kubernetes namespaces (which would explode namespace counts), Agent Substrate uses a lightweight "Atespace" concept stored in Redis. Atespaces are the isolation boundary and the first half of an actor's DNS address. They're cheap to create (no Kubernetes object overhead), making them practical at billion-actor scale.

### mTLS everywhere, with short-lived pod certificates
The `podcertcontroller` issues short-lived certificates to every component (ateapi, atelet, ateom). The atelet's credential broker then proxies identity tokens to worker pods via a Unix socket restricted to the same node. This means worker pods never have long-lived credentials baked into their images.

### Network-triggered wakeup
The atenet router pauses inbound HTTP requests and calls `ResumeActor` before forwarding — treating the network itself as the wakeup signal. This avoids the need for a separate notification system and means actors appear always-online to clients, with the latency cost absorbed by the request's TLS handshake time.

---

## Comparison Notes

**vs. [[Security and Sandboxing]]'s sandbox taxonomy**: Agent Substrate operates at the platform-sandboxing layer (Kubernetes + gVisor/micro-VM), comparable to OpenSandbox and Navaris. But where those are general-purpose sandbox platforms, Substrate specializes in the **agent multiplexing** use case — the suspend/resume cycle is the product, not a feature. OpenSandbox gives you a sandbox; Substrate gives you a system that manages thousands of sandboxes' lifecycles automatically.

**vs. [[How We Contain Claude]]'s gVisor discussion**: Anthropic's containment postmortem uses gVisor as one of three isolation patterns for a single Claude instance. Agent Substrate uses the same gVisor checkpoint/restore for an orthogonal purpose — not just isolation, but **density**: packing hundreds of sandboxed agents onto a handful of physical hosts by hibernating idle ones. The same technology, different axis: Anthropic optimizes for security, Substrate optimizes for density.

**vs. [[Best Infrastructure Platforms for Coding Agents in 2026]]'s Firecracker convergence**: The industry is converging on Firecracker for sandbox isolation (Modal, E2B, Daytona, etc.). Agent Substrate's micro-VM class uses Cloud Hypervisor (not Firecracker), chosen for its `userfaultfd` demand-paging support — a technique that lets a VM "boot" instantly and fault in memory pages lazily, which is critical for the sub-second resume target. Firecracker's live snapshot is noted but current implementations share the memory image upfront, which is slower for the multiplexing use case.

**vs. [[Building Agents That Don't Break Themselves]]'s copy-on-write checkpointing**: Daniel Botha's pattern (durable reasoning loop, disposable execution sandboxes with CoW checkpointing) converges on the same checkpoint/restore primitive but at a different scale. Botha's approach is per-agent, in-process; Agent Substrate does it at the cluster level, with a control plane managing state across nodes. The difference is the difference between a local snapshot tool and a production orchestration layer.

**vs. serverless platforms (Cloudflare Workers, AWS Lambda)**: Traditional serverless is stateless — functions start cold or warm, but don't carry state between invocations. Agent Substrate is "serverless for stateful agents": the agent's RAM and filesystem survive across suspend/resume cycles, just like a hibernated laptop. This is a genuinely different compute model that no major cloud provider offers as a managed service.

---

*Sources: [[raw/substrate]], [[summary/substrate]]*
*Last updated: 2026-08-08*
