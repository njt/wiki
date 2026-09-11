# You Should Be Using Rootless Containers

Miguel Grinberg's case that Docker's root-owned daemon is a structural security hole, and that the daemonless Podman is the practical fix — with an honest accounting of what the switch actually costs. The argument is aimed at Linux users running Docker for convenience, but it lands squarely on the same problem the agent-sandboxing world is grappling with: any long-lived privileged daemon is an attack surface, and every convenience layer (the `docker` group, the always-on API) widens it.

---

## Key Quotes

> "Because the Docker daemon runs as the `root` user, an attacker that gains access to it can execute code with root permissions. You may think that this is just theoretical... Well, think again."

The core mechanism. Docker's client–server architecture means the daemon is the single privileged choke point; the socket is the key that opens it. Grinberg's demo is the proof — `docker run -v /:/host alpine:latest ls /host/etc/sudoers.d/` mounts the host filesystem and reads root-only files with no password, no prompt, nothing.

> "The truth is that using Docker only as the root user is incredibly inconvenient, so much that the details on how to reconfigure it to work without `sudo` are documented in the official Docker website, in spite of it being a security nightmare."

The subtle point that makes this a *real* vulnerability rather than a textbook one. The default install is safe only because it's unusable; the documented convenience fix (adding yourself to the `docker` group) is exactly the step that reopens the hole. Security and usability are pulling in opposite directions, and Docker's own docs push users toward the insecure side.

> "I find the rootless Docker solution a bit hacky. The official installation instructions ask you to do a regular install, then manually disable the Docker daemon, and finally run a script they provide that creates a user-level daemon replacement."

> "If you get used to typing `podman` instead of `docker`... the vast majority of the workflows will work the same, but all your containers will run under your own user, with no viable path for a root escalation."

The contrast Grinberg actually cares about: Docker's rootless mode is a retrofit bolted onto the daemon architecture, while Podman removes the daemon entirely. The daemon isn't just a process — it's the entire attack surface, and Podman deletes it.

> "if there is no daemon, how do applications launch or interact with containers through an API? This is actually a very popular way to use Docker."

The honest cost of daemonless design. No daemon means no auto-restart after reboot (you need `podman quadlet` + systemd) and no API unless you explicitly run `podman system service`. Grinberg doesn't hide that "most workflows" is doing a lot of work — some workflows genuinely need the daemon back.

---

## Key Themes

- **#concept** — Rootless containers as least privilege applied to container runtimes: the container's root is *your* user, not the host's root
- **#concept** — The daemon as attack surface: a long-lived, root-owned, always-on process is a privilege-escalation target regardless of how well-written it is
- **#pattern** — Daemonless architecture: remove the background service, and you remove the thing an attacker can reach
- **#tool** — Docker (client–server, root daemon) vs. Podman (daemonless, user-level, drop-in `docker` alias)
- **#comparison** — Convenience vs. security: the `docker` group, the default registry, the always-on API are all convenience layers with security costs

---

## Critical Analysis

The article is strongest where it names the *economic* nature of the problem: the insecure configuration isn't a mistake users make, it's the configuration Docker's own documentation steers them toward. That reframes the blame from "users are careless" to "the tool's default incentives point at the insecure option." It's the same shape as [[Security Is Hard, Y'all]]'s observation that when the system makes the safe choice indistinguishable from the unsafe one, users can't be expected to sort it out.

The Podman recommendation is right but softer than it sounds. Grinberg concedes "vast majority," and the catch list — fully-qualified image names, no reboot persistence without systemd, no API without a manual service — is the real content. Podman fixes the *security* problem cleanly but re-introduces, one piece at a time, everything the daemon gave you for free, now as opt-in work. That's a defensible trade (you should have to opt *in* to the dangerous thing), but it means the migration story is "relearn the workflow," not "alias docker=podman."

The macOS/Windows section is the weakest — Grinberg admits he's guessing about QEMU/Hyper-V internals and outsources the WSL question to commenters. One commenter (Joost M) supplies the missing rigor: WSL2 is "a mostly regular Hyper-V virtual machine," so it gets the same VM separation as Docker Desktop rather than the bare-kernel risk Grinberg worried about. The article's uncertainty here matters less than the admission that the author flagged it instead of papering over it.

For the agent-sandboxing context this wiki tracks, the relevance is direct. [[SmolVM]] sells itself on exactly this insight — "no daemon, the VMM is a library" — and Grinberg's demo is the mechanism that *explains why that matters*: a root daemon is a root-escalation path, and a library VMM has no such always-on target. Tools like [[cco]] that fall back to Docker as an isolation backend are inheriting this exact risk surface as their security boundary, which is worth sitting with. [[How We Contain Claude]] reaches the same conclusion from the other direction: write as little custom privileged code as possible, because it's where the incidents happen. A root-owned Docker daemon is a lot of custom privileged code.

---

*Sources: [[raw/you-should-be-using-rootless-containers]], [[summary/you-should-be-using-rootless-containers]]*
*Last updated: 2026-09-11*
