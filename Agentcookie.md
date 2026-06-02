# Agentcookie

Matt Van Horn's session state sync tool that continuously replicates browser cookies, API keys, and auth tokens from your primary Mac to the agent Mac over an encrypted Tailscale tunnel. The pitch: your agent should inherit your authenticated sessions, not re-establish them. Zero per-site auth ceremony.

---

## Key Quotes

> "your agent's session state, synced"

The tagline is the thesis. This isn't a general sync tool — it's purpose-built for a specific pain point: your agent Mac has no cookies, and pasting them manually is barbaric.

> "zero per-site auth ceremony"

The metric that matters. Every other approach (manual cookie-pasting, per-service API key provisioning, re-authenticating on the agent machine) requires the human to perform site-specific ritual. Agentcookie collapses all of that into a single pair-code exchange and then stays out of the way.

> "encrypted over Tailscale"

The trust model is explicit: this goes over your existing Tailnet, encrypted end-to-end with AES-256-GCM and per-peer keys derived from the pairing code. No cloud relay, no third-party custody of your cookies. The Tailnet-only binding (both ends only listen on tailnet-private addresses) is a genuine security decision, not a convenience shortcut.

---

## Key Themes

- **#tool** Agentcookie — macOS session sync: cookies + secrets bus over encrypted Tailscale
- **#concept** Agent-native auth — the agent inherits human session state rather than maintaining its own identity. Different from service accounts, different from OAuth
- **#concept** Zero-ceremony onboarding — the pair-code model means one human action enables all services. Compare to provisioning individual API keys per service
- **#pattern** Dual delivery surfaces — cookies for browser-driving agents, a secrets bus for CLI tools. Two mechanisms because browsers and CLIs consume auth differently
- **#pattern** Adoption manifest — `agentcookie.toml` as the declaration format for what secrets a CLI needs. The discover command auto-detects these. A standard for credential declaration, not just a config file
- **#person** Matt Van Horn — creator of [[Printing Press]] and Agentcookie. Building the infrastructure layer for agent-native computing

---

## Critical Analysis

Agentcookie is solving a real and underappreciated problem. Every person running an agent on a second machine has hit the cookie problem — you SSH in, try to use a CLI, and realize the agent has no sessions. The current workarounds (copy-paste cookies from Chrome's dev tools, maintain a parallel set of API keys, re-auth on every new machine) are all terrible. Agentcookie makes the problem disappear.

The design is refreshingly boring in the right ways. AES-256-GCM, per-peer keys, replay defense, Tailnet-only listeners, signed binaries, `install-beta.sh` that works headless over SSH. This is not a research project — it's infrastructure built by someone who has actually run agents on remote Macs and knows exactly which sharp edges need sanding.

The adoption manifest (`agentcookie.toml`) is the most interesting architectural decision. Rather than trying to auto-detect every possible credential location (which would be fragile and terrifying), Agentcookie asks CLI authors to declare what they need. This is the [[Printing Press]] philosophy applied to secrets: give tool authors a simple standard to adopt, and the ecosystem grows from network effects. The three-tier integration strategy (explicit, pp-cli-derived, legacy v1) shows they're serious about adoption, not purity.

The limitation is the macOS-only constraint. "macOS only on both ends today" is honest, but it means this doesn't help Linux agent servers or cloud VMs. The Tailscale dependency is also a constraint — if you're not on a Tailnet, agentcookie offers nothing. These are reasonable trade-offs for a tool built by one person solving their own problem, but they limit the addressable market.

Compared to [[Chrome DevTools MCP — Debug Your Browser Session]], which takes the opposite approach (bring the agent to your authenticated browser), Agentcookie brings the authenticated state to the agent's machine. These are complementary strategies: DevTools MCP for interactive debugging where the developer is present, Agentcookie for unattended agent operation on a remote Mac. The real power move would be using both: Agentcookie keeps the remote agent continuously authenticated, and DevTools MCP lets you occasionally inspect what it's doing.

The bigger question is whether this should exist at all, or whether agents should have their own identities. The counter-argument: giving an agent your cookies means it can do anything you can do on those services, which is the whole [[yolo-cage]] problem. Agentcookie's defense here is that the agent already has access to those cookies if it's browser-driving — the sync just makes it work on a different machine. But the blast radius question is real: if the agent Mac is compromised, the attacker gets a live set of session cookies for dozens of services.

---

*Sources: [[raw/agentcookie]]*
*Source URL: https://agentcookie.dev/*
*Last updated: 2026-06-02*
