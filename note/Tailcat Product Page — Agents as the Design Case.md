# Tailcat Product Page — Agents as the Design Case

Tailscale's official Tailcat page sells the same tool as the earlier GitHub ingest — the data plane (WireGuard + NAT traversal + DERP) with the control plane deleted — but the framing has shifted: this version is written for the age of agents. The headline use cases are "give an agent access to a test machine for one task" and "piping your files over tailcat" into an AI agent sandbox, with the FAQ answering directly why agents are the fit: they "can't always complete login flows and may not need persistent network access."

---

## Key quotes

> "Agents often need to reach a machine for a specific task, but they can't always complete login flows and may not need persistent network access. Give the agent a tailcat address and it can connect to the machine for that workflow without joining a tailnet."

This is the sharpest sentence on the page. Almost every agent-authentication story in this wiki ([[Machine-to-Machine Authentication]], [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]]) assumes the hard problem is issuing and revoking agent identity. Tailcat's answer is to skip identity issuance entirely: the address *is* the grant, single-use by design. It's a third option between "no access" and "identity in the IAM system."

> "Treat it like a password. Don't log it, commit it to source control, or share it with anyone who shouldn't have access. Definitely don't tweet it."

Bearer capability security, stated without apology. Note the tension the page never fully resolves: an address that can't complete login flows is precisely an address an agent can't lose in a login flow — and also precisely an address that will end up committed to source control by an agent writing its own notes. "Don't log it" is advice aimed at humans; agents leak by default.

> "Use Tailscale when you need a governable network. Use tailcat when you need a quick connection."

A clean product-line split, and honest about it: Tailcat "intentionally leaves out accounts, identity, policies, device management." The hosted relay "keeps metadata logs and isn't intended for privacy" — a rare admission on a product page that the bootstrap path is a privacy trade.

## Key themes

#tool #concept #pattern

## Analysis

Compared to the GitHub README version of the story (see [[Tailcat]]), this page is less architecture and more positioning — and the positioning is the news. The capability-address model has quietly become an *agent access primitive*: connection grants that need no identity provider because they expire with the task. That puts it in direct conversation with the wiki's agent-identity material, as a deliberate counterpoint rather than a complement. Where [[The Agent Access Model]] and Tailscale's own [[AI Agents Need Their Own Identity and Least-Privilege Access]] argue agents need real, revocable, least-privileged identities, Tailcat argues that for ephemeral single-task access, the cheapest safe unit is a disposable bearer secret — ephemeral keys by default, so revocation is just letting the process die.

The honest risks are the ones bearer tokens always carry: possession is the only proof, the address is confusable-free but copyable, and an agent with the address in its context can exfiltrate it like any other secret. The optional key-based allowlist is the escape hatch, but the default is trust-on-possession. My take: for human-to-human quick connections this is fine; for agent access it works exactly because it is scoped to one task and one machine — use it anywhere broader and you've reinvented the over-permissive access problem with a shorter string. The page's own "definitely don't tweet it" line is doing a lot of load-bearing work.

## Related pages

- [[Tailcat]] — the earlier GitHub-source ingest covers the wire-level architecture (magicsock, DERP, `tc…` address format) that this marketing page only gestures at; the two together form the full picture.
- [[AI Agents Need Their Own Identity and Least-Privilege Access]] — same company, opposite philosophy: identity-first agent networking versus this capability-first shortcut. Tailcat is Tailscale selling both sides of the argument.
- [[The Agent Access Model]] — the capability-address pattern is a concrete, minimal instance of the access-model problem this page of the wiki theorises.

---
*Sources: [[raw/tailcat-tailscale]], [[summary/tailcat-tailscale]]*
*Last updated: 2026-11-17*
