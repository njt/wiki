---
url: https://blog.miguelgrinberg.com/post/you-should-be-using-rootless-containers
title: "You Should be Using Rootless Containers"
author: Miguel Grinberg
date_fetched: 2026-09-11
date_published: unknown (not present in fetched text; post discusses Ubuntu 26.04, so ~2026)
topics:
  - security-and-sandboxing
---

# You Should Be Using Rootless Containers

Miguel Grinberg argues that Docker's security problems are structural, not incidental: Docker's client–server architecture requires a daemon that runs as `root`, and anyone who can reach that daemon — through the Docker socket — can execute code with root privileges. He demonstrates a passwordless root escalation on his own Ubuntu system with a single `docker run -v /:/host alpine:latest ls /host/etc/sudoers.d/` command that mounts the host filesystem inside a container.

The catch: the default Docker install is not itself the problem, because `docker` is normally only callable by root. The vulnerability appears the moment users relax that — most commonly by adding themselves to the `docker` group to avoid typing `sudo`, a convenience change Docker's own docs walk people through. Grinberg's guess is that almost every Linux Docker user has made this change, and thus is exposed.

His fix is rootless containers. Docker has a rootless mode, which he finds "a bit hacky" (regular install, then disable the daemon, then run a script that builds a user-level daemon). His preferred alternative is **Podman** — daemonless, so containers run under your own user with no background root service and no viable escalation path, and a drop-in `docker` replacement for most workflows. The costs are real but small: Podman sometimes needs fully-qualified image names (`docker.io/library/postgres`), won't restart containers after a reboot without `podman quadlet`/systemd, and only exposes an API if you explicitly start `podman system service` (which is Docker-API-compatible).

Grinberg also covers macOS/Windows (Docker runs in a VM, adding a layer of separation) and flags WSL as the exception — the Linux kernel runs directly on the host, so the same escalation risks plausibly apply. He now uses Podman and Podman Compose for most work, keeping Docker only for quick tests on disposable VMs.
