# Self-Hosted HTTP Tunnels with SSH and nginx

Vincent Bernat builds a self-hosted ngrok replacement out of parts he already has: OpenSSH's reverse port forwarding plus an nginx wildcard-subdomain proxy, with time-limited signed URLs for access control and a shell script that turns the whole ritual into one command. It is a small, complete demonstration of a particular engineering aesthetic — refuse the new dependency, compose the boring primitives you already run.

---

## The build

- `ssh -R 0:localhost:8080` makes the remote server allocate an ephemeral port and forward it to the local service.
- nginx matches `p<port>.ssh.luffy.cx` with a regex `server_name` and proxies to `127.0.0.1:<port>`. A wildcard Let's Encrypt certificate (DNS-01 via a Route 53 zone) closes the loop.
- **Access control** is the clever bit: nginx's `secure_link` module checks an MD5 hash over `expires + port + secret`. The hash rides in the URL as the basic-auth *username* — because base64 is case-sensitive and domain names are not — alongside the expiry timestamp. Bad hash → 401 with a `WWW-Authenticate` challenge; expired → 410.
- A helper script finds the ephemeral port by walking the process tree for `sshd-session` ancestors and reading their listening sockets with `ss`, then prints tokenised URLs. Wired up as an `ssh RemoteCommand`, the user experience is one command.

> "With this solution, I only rely on OpenSSH and nginx, two pieces of software already running on this server."

That sentence is the thesis. The trade is explicit: more cleverness per component, zero new components.

> "I suppose you are now saying: 'Vincent, this is not very convenient! I'll stick with ngrok if you don't mind.' Okay, I hear you. Let's write a helper script."

A nice piece of rhetorical honesty — he concedes the raw mechanism is unusable by hand, then absorbs the ergonomics cost into a script rather than a dependency.

## Themes

#tool #concept #pattern

## Analysis

The post is really an argument about **dependency budgets** dressed as a tutorial. The tunneling space is crowded — ngrok, Cloudflare Quick Tunnels, frp, localtunnel, sish — and each option buys convenience with either a third-party service or a new server-side component. Bernat's version has a real cost: hashed secrets in MD5 (`secure_link` is not cryptographic-grade), an approach to auth that is charmingly hacky, and a helper script that greps process tables — something a distro update could quietly break. But it rides on two daemons that were already running, already patched, already monitored. That is the trade worth naming: not "self-hosted vs SaaS" but *new surface vs known surface*.

There is also a lesson in where the difficulty actually lived. The forwarding and proxying were trivial; the hard problems were **URL-safety of base64 in a case-insensitive hostname** and **discovering a port OpenSSH refuses to expose in any environment variable**. Integration friction, not the core mechanism, is where the engineering went — a pattern that recurs in every "just glue two existing tools" project.

## Relation to the wiki

- Complements [[Tailport]] directly: both solve "expose a local port safely", but Tailport builds a Go TUI around Tailscale's CLI while Bernat composes stock daemons with zero new code beyond a shell script — the minimal-dependency mirror image of the same problem.
- Strengthens [[Ditch the Token Headache — SSH Just Works]]: both make the same bet that SSH is the durable, configure-once substrate for developer plumbing, and both treat credential lifetime as the central design question — here the signed URL *expires* precisely where Kuehn's keys never do.
- Adds a concrete case to [[Security and Sandboxing]]: access control implemented inside nginx with signed, expiring capability URLs rather than an auth service — a capability-token pattern at its smallest viable scale.
- Sits naturally beside [[The GUS Stack — Go, Unix, SQLite]] in spirit: build from a small set of boring, permanent primitives instead of adopting a new service per problem.

---
*Sources: [[raw/2026-http-over-ssh]], [[summary/2026-http-over-ssh]]*
*Last updated: 2026-10-10*
