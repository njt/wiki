# OpenTunnel

OpenTunnel is a CLI and SDK for creating end-to-end-encrypted public URLs for services running on localhost — an ngrok-class tool with a zero-knowledge relay: the relay routes by TLS SNI but the traffic stays encrypted until it reaches your machine, which holds the only private key.

---

The pitch is three lines: `opentunnel route add 47365`, and your localhost service is reachable from anywhere at a generated public URL. Underneath, the design is careful. The private key is generated on your machine and never leaves it; the relay requests a certificate for your tunnel hostname but sees only the public half, and renews it for the same key. A persistent encrypted bridge connects your machine to the relay; visitors' TLS streams pass through the relay opaquely (it reads only the hostname from the handshake) and terminate on your box. The SDK lets multiple apps share one tunnel, each with its own subdomain route — the examples are agent-flavoured (a service called "opencode", webhooks on :8080).

The honesty is what stands out. The privacy page is a threat model, not marketing: it tells you the relay *can't* read your traffic, but also that tunnel hostnames are effectively public (published to CT logs the moment they're created), that wildcard certificates mean route names never appear in logs but *are* guessable, and that the tool offers no authentication at all — "put auth in the services themselves."

---

## Key quotes

> "the relay can't read your traffic … the bytes stay encrypted until they reach your machine. the relay has no key to decrypt them."

The core architectural claim, and the differentiator versus naive tunneling services that terminate TLS at the edge. Routing-by-SNI-without-termination is the same trick Cloudflare and Tailscale use in various forms.

> "anyone with your url can reach your services. when a tunnel is created, its certificate is published to certificate transparency logs, so the hostname is discoverable by anyone watching them."

A rare admission for a product page: your "secret" URL is discoverable infrastructure-wide via CT logs. This is the honest counterweight to the convenience pitch.

> "put auth in the services themselves / anything sensitive behind a tunnel should authenticate on its own."

The tool deliberately does nothing about access control. Good doctrine — and it echoes the wiki's recurring least-privilege theme that the perimeter is never the security boundary.

## Key themes

#tool #concept #pattern

## Analysis

This is a small tool with an unusually mature privacy posture. Most tunneling products monetise exactly the trust OpenTunnel refuses to take: terminating TLS at the relay gives them (and any attacker who breaches them) your plaintext. Building the relay as a blind router is the right call, and documenting the CT-log leak of hostnames rather than hiding it is the kind of threat-model candour that should be table stakes and isn't.

The obvious caveat: a blind relay still sees *metadata* — who is connecting to which tunnel, when, from where, and traffic volume. "End-to-end encrypted" should not be read as "the relay knows nothing." And the wildcard-cert design, while tidy, means your route namespace is only as private as its unguessability — "don't treat them as authentication" is doing a lot of work for a tool whose URL *is* the only gate.

The agent angle is implicit but interesting: the SDK's shared-tunnel, per-service subdomain model fits the current wave of agent runtimes that spin up local servers (opencode, webhooks, preview UIs) and need to expose them for callbacks, OAuth flows, or human review from another device. It is plumbing for the same pattern [[Ditch the Token Headache — SSH Just Works]] and [[Self-Hosted HTTP Tunnels with SSH and nginx]] cover, done with a crypto-clean relay instead of raw SSH or self-hosted nginx.

This strengthens [[Self-Hosted HTTP Tunnels with SSH and nginx]] — Bernat's DIY build achieves shareable localhost URLs but terminates TLS at the proxy, which OpenTunnel's relay design makes unnecessary; it nuances [[Tunnet]], which bundles tunneling into a mesh VPN where the relay is trusted infrastructure rather than a blind router; and it connects to [[Zero Trust for AI Agents]] on the point that tunnels are transport, not authorization — the service itself must hold the authz decision.

---
*Sources: [[raw/opentunnel-xyz]], [[summary/opentunnel-xyz]]*
*Last updated: 2026-10-18*
