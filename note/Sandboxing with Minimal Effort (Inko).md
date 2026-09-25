# Sandboxing with Minimal Effort

Yorick Peterse adds an in-process sandboxing API to Inko, wrapping Landlock, Seatbelt, Capsicum and pledge/unveil behind one small interface, and uses it to argue a design thesis: the value of a security feature lies in its ease of use, not its capability.

---

## What it says

Memory safety only goes so far — "the code is still written by developers and developers are, by and large stupid, myself included." Containers (he shows his shost Podman quadlet: dropped capabilities, read-only volume, user namespace mapping) are a good outer layer, but the better design is for the application itself to restrict its own capabilities regardless of how it runs.

The API is tiny. Sandboxing shost took:

```inko
fn enable_sandbox(config: ref Config) {
  let s = Sandbox.new
  match config.tls {
    case Some(v) -> s.path(v, read)
    case _ -> {}
  }
  s.path(config.sites.path, read)
  s.tcp(config.port, bind)
  s.enable
}
```

Everything not allowed is denied. The article walks the trade-offs candidly: FreeBSD support is a deliberate no-op because Capsicum requires restructuring programs around pre-opened resources and `openat` (possibly libcasper), while Landlock and Seatbelt can be applied without changing program structure. Platform quirks leak through anyway — under Landlock you must also authorise the ELF program interpreter and any shared libraries in non-standard locations.

## Key quotes

> "the value of a security feature lies not in what it can do, but rather in it's ease of use"

This is the load-bearing sentence. Security features fail in practice through non-adoption, and the way to win is to make the safe path the lazy path — the same instinct behind making a sandbox so cheap you'd add it even where a container already exists ("there's no reason *not* to use it").

> "using an LLM to do the writing instead doesn't improve things; if anything it makes it even worse given the average LLM has the intellect of a talking parrot with a bad drinking habit"

Tossed off in passing, but it lands the real motivation: as more code is written by agents, defence-in-depth that costs nothing stops being optional. His container-plus-in-process-sandbox layering is exactly the assumption agent-sandboxing work makes, arrived at from the application side.

## Themes

#concept (defence in depth, capability restriction) #pattern (make the secure path the easy path) #tool (Inko sandbox API, Landlock, Seatbelt, Capsicum, pledge/unveil)

## Analysis

The honest part of this piece is the FreeBSD no-op. Most API designers would have shipped a leaky Capsicum shim and called it cross-platform; Peterse instead documents that Capsicum's capability model (close the world at `cap_enter`, open everything relative to pre-opened directory handles) is a *program architecture*, not a flag, and refuses to pretend otherwise. That's a real taxonomy of OS sandboxing: retrofit-able restrictions (Landlock, Seatbelt, pledge) versus structural ones (Capsicum), and the Inko API silently abstracts the first class and punts on the second.

The weak spot is that "minimal effort" is measured on a well-behaved server. Anything dynamic — spawning subprocesses, loading plugins, writing temp files — will need a growing rule list, and the API offers no story for that. The argument also leans on the author's own taste ("I may be biased on account of, well, having written it"); the ease-of-use claim is backed by one ten-line example rather than adoption data. Still, the design principle is sound and travel-worn: it's the same reason `chmod` beats ACLs for most people.

## Related pages

This strengthens [[A Deep Dive on Agent Sandboxes]] with the application-side view: that piece treats the sandbox as harness infrastructure around the agent, while Peterse shows the program sandboxing *itself* — the same layering instinct, one level down.

It nuances [[Fences, not Sandboxes]]: Yegge argues OS-level containment becomes obsolete when models mature; Peterse's API assumes the opposite — that deterministic kernel-enforced denial is worth ten lines of code regardless of who wrote the program, human or LLM.

It complements [[Break Away — Unlimited Tokens and Containment]]: Harper's agent escaped a VM shim through a leftover credential precisely because containment layers were assumed rather than layered; Peterse's "no reason not to" doctrine is the cheap insurance that scenario lacked.

---
*Sources: [[raw/sandboxing-with-minimal-effort]], [[summary/sandboxing-with-minimal-effort]]*
*Last updated: 2026-09-25*
