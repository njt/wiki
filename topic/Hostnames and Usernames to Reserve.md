# Hostnames and Usernames to Reserve

A comprehensive operational security checklist for any platform that lets users register names: which hostnames, email addresses, and URL paths must be blocked to prevent protocol-level attacks. The unifying principle is that **Internet protocols assume a human administrator approves every name** — automation breaks that assumption, and attackers rush into the gap.

---

## The Core Insight

Geoffrey Thomas identifies a single failure mode across three attack surfaces: when name registration is automated, an attacker can claim the very names that protocols trust most. CA/Browser Forum rules let `hostmaster@example.com` validate domain ownership for certificates. WPAD lets the owner of `wpad.example.com` proxy all web traffic. Same-origin policy means `example.com/admin` and `example.com/user1` share a security boundary.

> "Many Internet protocols make the assumption that a domain is manually managed by its owners."

This is the article in one sentence. DNS, email validation, and cookie domains all encode trust assumptions from an era when someone had to file a ticket to create a subdomain. Automated registration pulverizes those assumptions.

## Hostnames: The Unqualified Lookup Problem

When `a.example.com` looks up `b`, DNS resolution finds `b.example.com`. This means any user who registers certain names can intercept traffic meant for infrastructure.

| Name | Attack |
|------|--------|
| `wpad` | Web Proxy Auto-Discovery — owner becomes the proxy for all web traffic on the network |
| `isatap` | IPv6 tunnel autodiscovery — same proxy risk for IPv6 traffic on Windows |
| `autoconfig` | Thunderbird queries this for email server settings — password harvesting vector |
| `www` | Browsers prepend this when the bare domain doesn't resolve |
| `imap`, `pop`, `pop3`, `smtp`, `mail` | Email clients guess these server names |
| `localhost`, `localdomain`, `broadcasthost` | Hard-coded in `/etc/hosts` files worldwide |

The recommended hostname regex is `/^[a-z]([a-z0-9-]*[a-z0-9])?$/`. Dots should be blocked because they break wildcard certificates and cross-subdomain cookies. All-numeric usernames conflict with UID-based tools.

> "someone who owns this (unqualified) name can act as a proxy for all web traffic"

## Email Addresses: CA Validation Is the Sharp Edge

This is the most operationally urgent section. Certificate Authorities are required to validate domain ownership by emailing one of five addresses:

- `admin`, `administrator`, `webmaster`, `hostmaster`, `postmaster`

The examples are damning: a researcher registered `sslcertificates@live.com` in 2008 and obtained a certificate for `login.live.com`. In 2015, a Finnish IT professional did the same with `hostmaster@live.fi` and got a Microsoft certificate. If your platform allows anyone to register `admin@`, you're handing out TLS certificates for your domain.

Additional names to block (from RFC 2142 and Mozilla's bug tracker): `root`, `info`, `ssladmin`, `ssladministrator`, `sslwebmaster`, `sysadmin`, `is`, `it`, `mis`, `marketing`, `sales`, `support`, `abuse`, `noc`, `security`, `usenet`, `news`, `uucp`, `ftp`, `mailer-daemon`, `nobody`, `noreply`, `no-reply`.

> "these names are extremely unlikely to be used by legitimate users anyway"

Translation: this costs you nothing and prevents catastrophe. The only reason not to do it is not knowing you need to.

## URLs and the Same-Origin Trap

For platforms where usernames appear at the path level (`twitter.com/username`), user content shares an origin with your login page, your admin panel, and your API. Same-origin policy means they can read each other's data and manipulate each other's DOM.

> "these web pages can freely interact with each other and mess with each other's content"

Service workers make this worse — a single registered service worker from a user-controlled path can intercept all requests on the domain.

**Reserved paths** (all naturally blocked by the no-dots hostname rule): `robots.txt`, `favicon.ico`, `crossdomain.xml`, `clientaccesspolicy.xml`, `.well-known` (RFC 5785 — used by Thunderbird, Let's Encrypt ACME, and more).

## The Public Suffix Solution

The public suffix list (publicsuffix.org) solves the cookie scoping problem. By listing your domain as a public suffix, you prevent any page from setting cookies scoped to the bare domain — blocking supercookies and session-fixation attacks.

> "by making `example.com` a public suffix, nobody, not even code on `example.com` itself, can set a cookie for `example.com`"

This is the architectural fix: run your app on `www.example.com`, redirect the bare domain, and get listed on the public suffix list. User content goes on `username.example.com` — separate origins, separate security boundaries, safe.

## Critical Analysis

**What holds up:** The core vulnerability class is timeless because it's baked into protocols, not implementations. WPAD hasn't been redesigned. CAs still validate via those five email addresses. Same-origin policy is fundamental to web security. This article will remain relevant as long as these protocols exist — which is to say, indefinitely.

**What's dated:** Published November 2015, it predates Let's Encrypt's ACME (which added `dns-01` and `http-01` challenges, expanding the attack surface), widespread DNSSEC adoption, CAA DNS records, and the explosion of OAuth/OIDC flows where redirect URI registration creates similar namespace problems. The core list holds up, but the threat model has expanded.

**The deeper lesson:** This isn't really about hostnames. It's about the gap between *protocol trust models* (which assume manual, deliberate naming) and *platform product decisions* (which want frictionless self-service registration). Every platform that lets users choose names makes this tradeoff. The ones that get breached are the ones that didn't know the tradeoff existed.

**What's especially good:** Thomas doesn't just list names — he explains *why* each one is dangerous, gives the protocol context, and provides the regex to enforce it. This is a checklist you can hand directly to an engineer.

**What's missing:** No discussion of internationalized domain names (IDN homograph attacks — `admin` with a Cyrillic `а`), no coverage of Unicode normalization in usernames, and no mention of the operational reality that many platforms discover these problems only after they have 50,000 users and can't rename anyone. The "just block them upfront" advice is correct but incomplete without guidance on how to fix this retroactively.

**The meta-point:** This article is on Thomas's personal blog with a CC-BY-SA license. It's been cited in countless internal engineering docs. It's the kind of knowledge that used to circulate as oral tradition among senior infrastructure engineers — and the fact that someone wrote it down, comprehensively, with protocol citations, makes it genuinely valuable public infrastructure.

---

#security #dns #namespace #checklist #protocol-design #concept

## See Also

- [[Security and Sandboxing]] — namespace enforcement as architectural security, not detection
- [[Designing a Passively Safe API]] — block dangerous names upfront, fail safely, don't hope
- [[Correct by Construction]] — whitelist approach: define what's valid, everything else is blocked
- [[Feedback Loop is All You Need]] — deterministic enforcement of name restrictions, not trusting registration flows
- [[Systems Ideas That Sound Good]] — "deferred security" anti-pattern: you can't rename usernames later
- [[An Illustrated Guide to OAuth]] — same-origin policy, redirect URI security boundaries
- [[You Dont Want Long-Lived Keys]] — same "build it right upfront" philosophy applied to credentials
- [[Cybersecurity Is Proof of Work Now]] — the counterpoint: this is architectural prevention, not compute spend
- [[claude-ctrl]] — enforcement over suggestion: block names, don't hope nobody registers them
- [[Portless]] — namespace management for local dev, same "names are infrastructure" principle
- [[yolo-cage]] — same "don't trust, constrain" philosophy at the agent level
- [[Smart Models Dumb Pipes]] — the "dumb pipe" here is the username validation regex
- [[AI Killing B2B SaaS]] — security gaps from rapid deployment without namespace planning

---
*Sources: [[summary/names-to-reserve]]*
*Last updated: 2026-05-14*
