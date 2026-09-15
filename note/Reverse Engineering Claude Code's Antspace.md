# Reverse Engineering Claude Code's Antspace

This note analyses AprilNEA's March 2026 reverse-engineering of Claude Code Web's runtime environment, conducted from inside a Claude Code session using only standard Linux tooling. The session uncovered the Firecracker microVM sandbox, a custom init/IPC system, an unstripped Go binary full of Anthropic internals, and — the headline — an undocumented deployment platform called Antspace that suggests Anthropic is building a full AI-native PaaS.

---

## The argument in one paragraph

Claude Code Web sessions run inside Firecracker microVMs restored from pre-warmed snapshots, supervised by a custom Rust init (`process_api`) that speaks a WebSocket process protocol — and the `environment-runner` Go binary shipped inside that sandbox is unstripped, carries full debug symbols from Anthropic's private monorepo, and contains a complete client for "Antspace", an internal deployment platform that has never been publicly mentioned anywhere, alongside a "Baku" web-app-builder environment whose default deploy target is Antspace rather than Vercel. The falsifiable claim: Anthropic is not merely an AI lab renting out inference, but is building a vertically integrated application platform — model, build runtime, database provisioning, and hosting — that will compete with Vercel, Replit, Lovable, and Supabase, with the structural advantage of owning every layer. If Antspace never ships publicly and Baku remains a claude.ai toy, that strategic reading is wrong.

---

## Key quotes

> Everything described here was discovered through standard Linux tooling (`strace`, `strings`, `objdump`, `go tool objdump`) running inside a Claude Code session. No exploits, no privilege escalation, no network attacks. The binary was sitting right there, unstripped, with full debug symbols.

The methodology is the quiet star here: no Ghidra, no decompilation, just `objdump -t` and struct-tag string extraction. It is also a reminder that anything you ship inside an agent's sandbox is readable by the agent's user.

> A search for "Antspace" across the entire public internet turned up nothing: Anthropic's website, GitHub, blog, documentation, LinkedIn, job postings, conference talks, patent filings. **Zero results.** This platform has never been publicly mentioned anywhere.

A genuinely undocumented system found in the wild — rare in 2026, when everything leaks via job postings and npm packages. The zero-results claim is itself checkable, which is what makes it strong evidence rather than vibes.

> The fact that Anthropic built a full deployment protocol from scratch, rather than just wrapping Vercel's API, signals this is a strategic platform investment, not a quick integration.

This is the load-bearing inference of the whole piece. Building a tar.gz-upload, NDJSON-streaming deploy protocol is real engineering; you don't do that for an experiment you plan to abandon. It's an inference from effort, not from statements — reasonable, but not proof of intent to launch publicly.

> Shipping an unstripped binary with full debug symbols to production is... a choice.

Dry understatement doing a lot of work. The entire article exists because of one build-pipeline oversight; a stripped binary would have reduced this to guesswork.

> What's clear is that Anthropic's ambitions extend far beyond being just an LLM and AI agent company. They're building the infrastructure for a world where applications are spoken into existence, and they want to own every layer of that stack.

The thesis statement. "Spoken into existence" is doing rhetorical work — the evidence supports "Anthropic hosts what Claude builds on claude.ai," which is narrower than a general PaaS play, though not by much.

---

## Critical analysis

The non-obvious technical content is the snapshot architecture. Sessions don't boot; they resume from a VM frozen at a "SNAPSTART_READY" checkpoint, with block devices hot-swapped at restore time — a placeholder rootfs swapped for the session's ext4, plus squashfs overlays for the Claude Code tooling. The restore path is a checklist of distributed-systems hygiene: drop page caches (stale template cache returns garbage), reseed the CRNG (snapshot forking breaks crypto predictability), fix the wall clock via `clock_settime`, drop `CAP_SYS_RESOURCE`. The 48.5-hour gap between template creation and restore, and an ext4 mount count of 11, are lovely forensic details — the image is a shared template reused across sessions, which is exactly why the cache and RNG resets matter.

What's weak: the strategic leap from "deployment client exists in a binary" to "vertically integrated PaaS strategy" is bigger than the author admits. The version string is `staging-`-prefixed; Antspace may be internal infrastructure for Baku that never becomes a product, and the competitive framing (versus Vercel, Replit, Supabase) is the author's extrapolation, not Anthropic's. There's also an unexamined tension: the piece treats root access inside the VM as unremarkable, but the security table (JWT auth, `--block-local-connections`, token scrubbing) suggests Anthropic cares a lot about what escapes the sandbox — and yet the exfiltration channel here was simply "read the binary with your eyes." The defence wasn't broken; it was never aimed at this threat.

What's left out: any legal or ethical discussion of publishing reverse-engineered proprietary internals. The author is careful ("no exploits"), but publishing wire protocol specs and package trees from a private monorepo is a decision the article narrates without interrogating. Also absent: whether the same environment-runner ships in the CLI product, and what the BYOC mode implies about enterprise demand for running agent sessions on customer infrastructure — that's arguably the most commercially interesting finding and gets only a table.

---

## Related

- [[A Deep Dive on Agent Sandboxes]] — That page surveys sandbox designs for agent execution in general; this source is a concrete, dissected instance of the Firecracker-microVM approach, confirming with real internals (snapshot restore, cgroup-per-process, capability dropping) what that page describes at the pattern level.
- [[How We Contain Claude]] — That page argues for layered containment of Claude Code's blast radius; this source nuances it by showing what Anthropic's own containment actually looks like from the inside — and complicates it, since the biggest leak was an unstripped binary sitting inside the trusted boundary.
- [[Best Infrastructure Platforms for Coding Agents in 2026]] — That page maps the vendor landscape for agent execution infrastructure; this source complicates its picture by revealing that the biggest player's platform (Antspace) is one no survey could list, because it was completely undocumented at the time.
- [[Claude Code]] — That page collects Claude Code product knowledge; this source strengthens it with the server-side half of the story — the Web runtime, snapshot lifecycle, and Baku environment — that CLI-focused write-ups never touch.

---
*Sources: [[raw/reverse-engineering-claude-code-antspace]], [[summary/reverse-engineering-claude-code-antspace]]*
