---
url: https://gyrovague.com/2023/08/05/archive-today-on-the-trail-of-the-mysterious-guerrilla-archivist-of-the-internet/
archive_url: https://archive.ph/UNq2w
title: "archive.today: On the trail of the mysterious guerrilla archivist of the Internet"
author: Jani Patokallio (jpatokal)
date_fetched: 2026-05-15
date_published: 2023-08-05
---

# archive.today: On the trail of the mysterious guerrilla archivist of the Internet

By Jani Patokallio, August 5, 2023, on Gyrovague

## What It Is

An OSINT investigation into who runs archive.today (originally archive.is, also archive.md, archive.ph, etc.) — a widely-used web archiving service that, unlike the Internet Archive's Wayback Machine, has no opt-out mechanism and stores everything permanently with limited exceptions for law enforcement and illegal content.

## Origins & Domain History

- Domain archive.is registered May 16, 2012 by "Denis Petrov" from Prague, Czech Republic
- Archive.today domain followed in 2014; numerous variants (archive.li, .ec, .vn, .ph, .fo, .md) registered since
- "Denis Petrov" is a common Russian name — likely an alias
- Same contact info used for sketchy domains: carding forums, piracy sites, many with German keywords
- Dead ends: denispetrov.com, denis.biz, petrov.net all led nowhere useful

## The Masha Rabinovich Lead

- Stack Exchange investigator Ciro Santilli linked an archive.today LinkedIn-scraping account's profile picture to "Masha Rabinovich" in Berlin
- "masharabinovich" appeared in a 2012 F-Secure forum post complaining about archive.is being blacklisted
- Same username on Wikipedia: told off for adding excessive archive.is links, used Czech ISP fiber.cz
- Early Wikipedia edits included "Russian passport" and "Belarusian passport" pages
- Masha = common Russian diminutive of Maria; Rabinovich = Ashkenazi Jewish surname

## The volth Connection

- Early GitHub captures from archive.today linked to a now-deleted account "volth"
- Fluent Russian speaker, contributed extensively to NixOS (which archive.today uses for its infrastructure)
- Domain volth.com dates to 2004 but original Espinosa family owners appear unrelated

## Infrastructure

- Runs Apache Hadoop + Apache Accumulo on HDFS
- Text content replicated 3x across two European datacenters; images 2x
- At least one datacenter hosted by OVH (France)
- 2012: ~10 TB, ~€300/month
- 2014: ~€2,000/month
- 2016: ~$4,000/month
- 2021: ~500 million pages archived (~1,000 TB; Internet Archive is ~40,000 TB by comparison)
- Scraper uses a modified version of Chrome, cycling through a botnet of IP addresses to evade blocking
- Paywalled content accessed via logins obtained by "unclear means" (constant replenishment needed — once publicly asked for Instagram credentials)
- Perpetual domain troubles: operator predicted "one trouble with domains per year and each fifth trouble will result in domain loss"
- Users currently redirected to archive.md

## Funding

- As of 2021: ads + donations covered <20% of expenses; donations ~€6,000
- PayPal deactivated ~2022 (creator couldn't top up account — implying Russian location)
- Creator complained about cross-border payment difficulty across "the Iron Curtain"
- Current donations: Liberapay and BuyMeACoffee
- Creator skeptical of cryptocurrency — not supported
- Ads: Yahoo network, mobile only (not desktop)
- On good days ads "almost cover expenses"; on bad days, kicked out of ad networks due to NSFW archived content
- Contradiction: "almost cover expenses" vs "<20% of expenses" — suggests significant undisclosed income

## Operator Profile (Author's Assessment)

- One-person labor of love
- "A Russian of considerable talent and access to Europe"
- Constantly fighting: domain registrars, anti-scraping mechanisms, copyright enforcement, skittish advertisers, international sanctions on Russian citizens
- Has a second source of "considerable income that's likely somewhat sketchy"
- By staying anonymous, avoided legal battles like Sci-Hub's Alexandra Elbakyan faced
- Creator acknowledges the site is a "weak tool" that is "doomed to die"
- Bus factor of one; semi-legal status means no foundation or legal entity can ensure continuity

## Conclusion

The author characterizes the site as a "one-man battle against entropy" — thanking "Denis/Masha/whoever" for over 10 years of persistence and linking to BuyMeACoffee. All images in the post feature the Bibliotheca Alexandrina in Alexandria, Egypt.

## Comment

One anonymous commenter notes btdig.com still exists and is run by the same person.

Tags: archive, archive.is, archive.md, archive.today, copyright, internet, internet archive, library
