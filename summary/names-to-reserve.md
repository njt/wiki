---
url: https://ldpreload.com/blog/names-to-reserve
title: Hostnames and Usernames to Reserve
author: Geoffrey Thomas (geofft)
date_fetched: 2026-05-14
date_published: 2015-11-26
license: CC-BY-SA
topics:
  - security-and-sandboxing
---

# Hostnames and Usernames to Reserve

Core argument: automatically registering usernames usable as hostnames, emails, or URL paths is dangerous because "many Internet protocols make the assumption that a domain is manually managed by its owners." When names like `admin` can be grabbed by anyone, security breaks down.

## Hostnames

Problems arise from **unqualified lookups**: when `a.example.com` looks up `b`, it finds `b.example.com`.

| Name | Risk |
|------|------|
| `localhost`, `localdomain`, `broadcasthost` | Present in `/etc/hosts`; hard-coded assumptions |
| `www` | Browsers prepend this if domain itself doesn't resolve |
| `wpad` | Web Proxy Auto-Discovery; owner can proxy all web traffic |
| `isatap` | IPv6 tunnel autodiscovery (Windows); proxy risk for IPv6 |
| `autoconfig` | Thunderbird queries for email settings — password harvesting |
| `imap`, `pop`, `pop3`, `smtp`, `mail` | Email clients guess these server names |

**Syntax restrictions:** hostnames must match `/^[a-z]([a-z0-9-]*[a-z0-9])?$/`. DNS is case-insensitive. Dots cause problems with wildcard certificates and cross-subdomain cookies. All-numeric usernames conflict with UID-based tools.

## Public Suffix List

Two pages with different origins cannot normally interact, but cookies are an exception — `www.example.com` and `login.example.com` can set cookies scoped to `example.com`. This creates supercookies and session-fixation. Solution: list your domain as a public suffix (publicsuffix.org). Then "nobody, not even code on `example.com` itself, can set a cookie for `example.com`." Run your site on `www.example.com` or a completely separate domain.

## Email Addresses

CA/Browser Forum Baseline Requirements mandate CAs use only these five when validating domains via email:
- `admin`, `administrator`, `webmaster`, `hostmaster`, `postmaster`

Additional safety reserves (Mozilla bug tracker):
- `root`, `info`, `ssladmin`, `ssladministrator`, `sslwebmaster`, `sysadmin`, `is`, `it`, `mis`

RFC 2142 service mailboxes:
- `info`, `marketing`, `sales`, `support`, `abuse`, `noc`, `security`, `postmaster`, `hostmaster`, `usenet`, `news`, `webmaster`, `www`, `uucp`, `ftp`

Automated process reserves:
- `mailer-daemon`, `nobody`, `noreply`, `no-reply`

## URLs

For platforms where usernames appear at top-level paths (e.g., `twitter.com/username`), restrict usernames as if they were hostnames. Reserved URL paths (all contain dots, thus invalid as hostnames):

- `robots.txt`, `favicon.ico`, `crossdomain.xml`, `clientaccesspolicy.xml`
- `.well-known` (RFC 5785 — Thunderbird, Let's Encrypt ACME, BrowserID, RFC 7711)

**Critical:** same-origin user content can interact freely. Service workers make same-origin attacks easy. Use separate hostnames within a public suffix — `user1.example.com` and `user2.example.com` are separate origins.

## Key Quotes

1. "Many Internet protocols make the assumption that a domain is manually managed by its owners"
2. "Browsers will often prepend this if the domain itself does not resolve as a hostname"
3. "someone who owns this (unqualified) name can act as a proxy for all web traffic"
4. "by making `example.com` a public suffix, nobody, not even code on `example.com` itself, can set a cookie"
5. "these names are extremely unlikely to be used by legitimate users anyway"
6. "the easiest approach is to restrict these usernames as if they were hostnames"
7. "these web pages can freely interact with each other and mess with each other's content"

Inspired by a GitHub issue for Sandstorm's sandcats.io dynamic DNS service. Thanks to Asheesh Laroia for review.
