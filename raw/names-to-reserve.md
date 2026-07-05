---
url: https://ldpreload.com/blog/names-to-reserve
date_fetched: 2026-07-05
backfilled: true
---

# Hostnames and usernames to reserve

If you're setting up a service where people can register their own usernames to be used as a hostname (`username.example.com`), email address (`username@example.com`), or URL path (`example.com/username`) within your domain, there are some common names you should avoid letting the general public register.

Many Internet protocols make the assumption that a domain is manually managed by its owners, and in particular assume that a name like `admin` must have been registered or approved by the actual owners. Automatic registration breaks this assumption, and has been the source of some attacks. Microsoft Live has fallen victim to this multiple times: in 2008, a researcher signed up for `sslcertificates@live.com` and used it to get a login.live.com certificate, and as late as this March, the same problem happened to live.fi, the Finnish version of the service, when an IT professional tried registering the email account `hostmaster@live.fi` as his personal Live account, and then found he could receive a certificate for that domain.

This is a list of all the names I know that should be restricted from registration in automated systems. If you know of others, please let me know and I'll update this page.

**tl;dr:** Regardless of how you're currently using usernames, restrict them to lowercase letters, digits, and hyphens, starting with a letter and not ending with a hyphen (that is, `/^[a-z]([a-z0-9-]*[a-z0-9])?$/` as an extended regex). Ban all the names in this file (last updated 2015-11-21). Get yourself listed as a public suffix: see below for directions and implications.

## Hostnames

Most of these problems involve a computer on the domain doing an unqualified lookup: when a computer named `a.example.com` looks for `b`, it will usually find `b.example.com`. If you're running a simple hosting service, or similar, you may not need to block all of these, but these names are extremely unlikely to be used by legitimate users anyway. So you may as well block all of them to allow expanding in the future.

- `localhost`- `localdomain`- `broadcasthost`- *usually*present in- `/etc/hosts`, and applications or scripts might hard-code an assumption about them having their usual value (especially for- `localhost`).
- `www`
- `wpad`
- `isatap`- `WPAD`and- `ISATAP`.
- `autoconfig`- `autoconfig.example.com`for settings when attempting to set up- `example.com`email. Good way to harvest passwords.
- Along those lines, `imap``pop``pop3``smtp``mail`

Note that valid hostnames are restricted in syntax: they must only contain letters, digits, or hyphens, and cannot start or end with a hyphen. DNS is case-insensitive, so make sure there are no case collisions. An older standard prevents hostnames from starting with a digit, which is a straightforward way to prevent all-numeric usernames (which can cause problems with tools that accept either names or UIDs). Dots separate portions of a domain name and cause various problems (wildcard certificates only apply to one level, `a.b.example.com` can read and write cookies for `b.example.com`, etc.), so they're usually more trouble than they're worth. *DNS records* are much more liberal, but names that don't follow these rules will generally not resolve as hostnames: you can look them up with `dig`/`host`/etc., but you can't use them in applications. Checking hostname syntax also prevents you from worrying about names like ** _tcp** or 

**, which are used in SRV records.**

`_udp`### Become a public suffix

Most parts of the web platform consider two pages with different *origins*, that is, scheme (`http` / `https`), hostname, and port number, to be unrelated websites that cannot interact with each other by default. However, there are a few exceptions, most notably cookies. Web pages at `www.example.com` and `login.example.com` are allowed to set cookies with a scope of `example.com`, despite not sharing the same hostname / origin. The simple rule of allowing parent domains created the problem of supercookies: `example.com` could set a cookie scoped to `.com`, which would then be sent to all sites ending in `.com`. There are two big problems with this: the first is privacy (being tracked across websites), and the second is session-fixation attacks, where an attacker can overwrite your session cookie with their own, and have your actions (including logging in or sending private data) happen within the attacker's session.

The immediate fix was to ban top-level domains, but this still allowed setting cookies for publicly-registrable suffixes like `.co.uk` that weren't at the top level. So browser vendors created the public suffix list to track which suffixes are open for public registration. The public suffix list now includes not only "ICANN" entries, such as `.com` and `.co.uk`, but also "private" entries, such as `.herokuapp.com` and `.github.io`, since the same problems exist with allowing users to set cookies for all Heroku or GitHub Pages users.

So, if you are letting users register hostnames in your domain, you should get it listed as a public suffix, which requires just sending a pull request or an email. It takes some time for the update to reach browsers (the list is compiled into browsers, so it's only updated by a browser version update), so you should try to do this as far in advance as possible before launching.

Note that by making `example.com` a public suffix, nobody, **not even code on  example.com itself**, can set a cookie for 

`example.com`. If you have a website of your own that needs cookies (analytics, registration, etc.), you'll need to run it at e.g. `www.example.com`, and make `example.com` just a redirect. Alternatively, you can use a completely separate domain for your own site vs. your users' sites, as with the Heroku and GitHub examples: their own websites are `heroku.com` and `github.com`.## Email addresses

The CA/Browser Forum Baseline Requirements, section 3.2.2.4 item 4, requires that if a CA is going to validate a domain by coming up with an administrative email address on its own, it may only use ** admin**, 

**,**

`administrator`**,**

`webmaster`**, or**

`hostmaster`**. Reserve all of those names, regardless of whether they go somewhere useful.**

`postmaster`All CAs are supposed to be compliant with that these days, but for safety's sake, also reserve ** root**, 

**,**

`info`**,**

`ssladmin`**,**

`ssladministrator`**,**

`sslwebmaster`**,**

`sysadmin`**,**

`is`**, and**

`it`**(see this 2009 comment on Mozilla's bug tracker).**

`mis`RFC 2142 defines the names `info`, ** marketing**, 

**,**

`sales`**,**

`support`**,**

`abuse`**,**

`noc`**,**

`security``postmaster`, `hostmaster`, **,**

`usenet`**,**

`news``webmaster`, `www`, **, and**

`uucp`**. You won't need most of these to actually reach a useful mailbox, though you should reserve all of them.**

`ftp`You may want to reserve ** mailer-daemon**, 

**(a default UNIX user account),**

`nobody`**,**

`noreply`**, etc. for automated processes that send email.**

`no-reply`Again, as these names are unlikely to be used by legitimate users, it's usually worth blocking them now and keeping your options open, even if you're not currently offering email service. You may add an email service in the future (Amazon launched Send to Kindle by email over a decade after introducing user accounts). As always, you can *manually* register these names to trusted or internal users.

## URLs

For many websites with user-provided content, like Twitter, Facebook, or GitHub, user-chosen usernames become part of the URL at top level (`https://twitter.com/geofft`, `https://github.com/geofft`). If you're building a website like this, the easiest approach is to restrict these usernames as if they were hostnames. This has two advantages: the first is that it's easy to launch a hostname-based system later (e.g. GitHub Pages now supports `geofft.github.io`) if you know that all your usernames are valid hostnames.

The second is that there are several URL paths you need to reserve at top level, and all of them happen to contain dots and are therefore invalid hostnames. If you do permit dots, you need to block the following names:

- `robots.txt`
- `favicon.ico`
- `crossdomain.xml`- `crossdomain.xml`.
- `clientaccesspolicy.xml`- `crossdomain.xml`.
- `.well-known`- `/.well-known/`.

(These are URLs, not filenames. You should of course also disallow users from creating files named e.g. `.htaccess` if your web server respects those.)

All of these are invalid hostnames, so simply requiring usernames to be valid hostnames avoids having to check for these specific cases. If you're only allowing users to choose some portion of the URL, and inserting other text (e.g., `example.com/user/geofft`, `example.edu/~geofft`), then you don't have to worry about this, but again it may still be useful to keep your options open for other URL, hostname, or email schemes in the future.

Do not allow users to publish custom HTML, *especially* not custom scripts, at these sorts of URLs. `https://example.com/user1`, `https://example.com/user2`, and `https://example.com/login` all share the same origin, so by the same-origin policy, these web pages can freely interact with each other and mess with each other's content. A few JavaScript interfaces, including service workers, make it very easy to attack another site on the same origin. If you want users to be able to publish custom HTML and JS, use separate hostnames within a public suffix. `https://user1.example.com` and `https://user2.example.com` are separate origins, and if you have made `example.com` a public suffix as mentioned earlier, you can safely let them publish custom scripts, since the sites are no more able to interact with each other than two separate `.com` websites could.

*This post was inspired by a GitHub issue for Sandstorm's sandcats.io dynamic DNS service; thanks to Asheesh Laroia for pointing me at that thread and reviewing a draft of this article.*
